---
title: Automating Data Updates; Streamlining Elastic Index Repopulation with Aliases and PowerShell Scripts
date: 2023-06-16
categories: [Development]
tags: [elasticsearch, development, scripts, powershell, devops]
comments: true
image:
  path: /assets/images/2023-06-16-scripts-elasticsearch-powershell_1.jpg
  alt: Automation process.
---

**Introduction**

In the world of dynamic data, keeping Elasticsearch indexes up-to-date is essential to maintain accurate and relevant search results. However, updating indexes can be a challenging task, especially when it involves automating the process to ensure that the latest data is always available. In this blog post, we'll delve into a powerful technique for achieving automated data updates using aliases in Elasticsearch with help of PowerShell scripts.

**Automated Data Updates and Index Repopulation**

Automating data updates in Elasticsearch is crucial for real-time applications that depend on current information. To achieve this goal, we'll employ the concept of using aliases to efficiently repopulate indexes with new data.

**The Process:**

1. **Initial Setup:** Let's assume we have an index named `country` that stores information about countries. We start with `country_v1`. The script below will recreate it even if the first version is not present.

2. **Create a New Index:** When it's time to update the data, create a new index with a version indicator. For instance, `country_v2`.

    _Assumption: Index mapping/settings file is present in this particular location "./Elasticsearch/country.json"_

    ```powershell
    param (
        [string]$ElasticUrl,
        [string]$Username,
        [string]$Password,
        [string]$IndexPrefix="country" # alias name, without version
    )
    
    $endpoint = "/_cat/aliases/$IndexPrefix"
    $aliasessUri = $ElasticUrl + $endpoint
    
    $headers = @{
        "Authorization" = "Basic " + [System.Convert]::ToBase64String([System.Text. Encoding]::UTF8.GetBytes( $Username + ":" + $Password))
    }
    
    $response = Invoke-RestMethod -Uri $aliasessUri -Method Get -Headers $headers
    
    $lines = $response -split '\r?\n'
    $sortedIndexNames = $lines | ForEach-Object {
        $splitData = $_ -split '\s+'
        $indexName = $splitData[1]
        if ($indexName -like $IndexPrefix + "_*") {
            $indexName
        }
    } | Sort-Object -Descending
    
    $lastIndexName = $sortedIndexNames | Select-Object -First 1
    
    if ($lastIndexName) {
        $startIndex = $lastIndexName.IndexOf("_v") + 2
        $version = $lastIndexName.Substring($startIndex)
        $nextVersion = [int]$version + 1
        $updatedIndexName = $lastIndexName -replace "_v$version", "_v$nextVersion"
    }
    else {
        $updatedIndexName = $IndexPrefix + "_v1"
    }
    
    Write-Host "If next Next index is present script will retry to create"
    
    Write-Host "Last index name : $lastIndexName"
    Write-Host "Next index name : $updatedIndexName"
    
    $indexUri = "$ElasticUrl/$updatedIndexName"
    $mappingFilePath = "./Elasticsearch/"+$IndexPrefix+".json"
    $mappingContent = Get-Content -Path $mappingFilePath -Raw
    
    $response = Invoke-RestMethod -Uri $indexUri -Method Put -Headers $headers  -ContentType "application/json" -Body $mappingContent
    
    if ($response.acknowledged) {
        Write-Host "Index '$updatedIndexName' created successfully."
    } else {
        Write-Host "Failed to create index '$updatedIndexName'."
        throw "Failed to create index '$updatedIndexName'."
    }
    
    ```

3. **Populate the New Index:** Index the new data into the `country_v2` index. Elasticsearch's indexing APIs can be used for this purpose.

4. **Switch Aliases:** Instantly switch the alias to point to the new index, making the new data active for queries.

5. **Alias Update:** Update the alias to point to the new index. The alias name, in this case, is still `country`, but it now points to `country_v2`.

    ```powershell
    param (
        [string]$ElasticUrl,
        [string]$Username,
        [string]$Password,
        [string]$UpdatedIndexName, # counttry_v2
        [string]$AliasName="country"      
    )

    $alias = "/_aliases"

    $uri = $ElasticUrl + $alias
    $headers = @{
        "Authorization" = "Basic " + [System.Convert]::ToBase64String([System.Text. Encoding]::UTF8.GetBytes( $Username + ":"+ $Password))
    }

    # Build the request body
    $requestBody = @{
        actions = @(
            @{
                add = @{
                    "index" = $UpdatedIndexName
                    "alias" = $AliasName
                }
            }
        )
    } | ConvertTo-Json -Depth 3

    # Send the request to create the alias
    $response = Invoke-RestMethod -Uri $uri -Method POST -Headers $headers  -ContentType "application/json" -Body $requestBody

    if ($response.acknowledged) {
        Write-Host "New alias created successfully!"
    } else {
        Write-Host "Failed to create new alias."
        throw "Failed to create new alias."
    }

    ```

6. **Remove Old Index Alias:** After the switch, remove the alias from the old index (`country_v1`).

    ```powershell
    param (
        [string]$ElasticUrl,
        [string]$Username,
        [string]$Password,
        [string]$LastIndexName, # country_v1
        [string]$AliasName # country     
    )
    
    if ([string]::IsNullOrEmpty($LastIndexName))
    {
        Write-Host "No index present to remove"
        exit 0
    }
    
    $alias = "/_aliases"
    
    $uri = $ElasticUrl + $alias
    $headers = @{
        "Authorization" = "Basic " + [System.Convert]::ToBase64String([System.Text.    Encoding]::UTF8.GetBytes( $Username + ":"+ $Password))
    }
    
    # Build the request body
    $requestBody = @{
        actions = @(
            @{
                remove = @{
                    index = $LastIndexName
                    alias = $AliasName
                }
            }
        )
    } | ConvertTo-Json -Depth 3
    
    # Send the request to remove the alias
    $response = Invoke-RestMethod -Uri $uri -Method POST -Headers $headers     -ContentType "application/json" -Body $requestBody
    
    if ($response.acknowledged) {
        Write-Host "Old alias removed successfully!"
    } else {
        Write-Host "Failed to remove old alias."
        throw "Failed to remove old alias."
    }
    ```

7. **Delete Old Index:** With no active alias, it's safe to delete the old index, freeing up resources.

    ```powershell
    param (
        [string]$ElasticUrl,
        [string]$Username,
        [string]$Password,
        [string]$Index # country_v1     
    )

    if ([string]::IsNullOrEmpty($Index))
    {
        Write-Host "No index present to remove"
        exit 0
    }

    $indexUri = "$ElasticUrl/$Index"

    $headers = @{
        "Authorization" = "Basic " + [System.Convert]::ToBase64String([System.Text. Encoding]::UTF8.GetBytes( $Username + ":" + $Password))
    }

    $response = Invoke-RestMethod -Uri $indexUri -Method Delete -Headers $headers

    if ($response.acknowledged) {
        Write-Host "Index $Index deleted successfully."
    } else {
        Write-Host "Failed to delete index $Index."
        throw "Failed to delete index $Index."
    }
    ```

**Benefits of Automated Index Repopulation Using Aliases**

1. **Real-time Updates:** Automated data updates ensure that the index is always up-to-date with the latest information, making search results accurate and relevant.

2. **Efficiency:** The alias mechanism allows for a seamless switch between indexes, preventing any significant downtime during the update process.

3. **Consistency:** Querying the same alias (`country`) guarantees consistent results, regardless of the underlying index being used.

4. **Scalability:** By automating the process, you can easily scale your application to handle large volumes of data without manual intervention.

5. **Rollback Capability:** In case of any issues with the new index, you can quickly switch back to the old index by updating the alias.

**Implementation Considerations**

1. **Data Processing Pipeline:** Design an automated data processing pipeline that fetches, transforms, and indexes new data as it arrives. The above-mentioned script can be incorporated into any CI/CD platforms, such as **Azure DevOps**.

2. **Monitoring and Alerting:** Implement monitoring and alerting mechanisms to ensure the automated process is running smoothly and to promptly address any potential issues.

3. **Scheduled Updates:** Schedule data updates based on the frequency of new data arrivals and the indexing capacity of your Elasticsearch cluster.

**Conclusion**

Automated data updates in Elasticsearch using aliases provide an efficient and reliable way to ensure that your indexes are always populated with the latest information. By seamlessly switching between indexes while maintaining a consistent alias, you can deliver accurate and real-time search results to your users. This approach not only enhances the user experience but also streamlines index maintenance and resource management, allowing you to focus on delivering the best possible search functionality to your application's users.
