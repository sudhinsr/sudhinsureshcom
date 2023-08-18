---
title: Updating and Using Release Variables in Azure DevOps Classic Release with PowerShell
date: 2023-08-15
categories: [Azure, DevOps]
tags: [devops, development, scripts]
comments: true
---

In the world of software deployment, efficient parameterization of your pipeline is key to success. Azure DevOps Classic Release provides a powerful way to manage these parameters, known as release variables. In this blog post, we'll explore how to not only update a release variable but also leverage it in subsequent steps using a PowerShell script.

## Enhancing Release Variables for Seamless Pipelines

Release variables allow you to pass dynamic values across stages and steps within a release pipeline. This becomes especially crucial when you need to maintain consistency and adaptability across various deployment environments.

## Scripting the Update and Usage of Release Variables

Let's dive into a PowerShell script that updates a release variable and showcases how it can be used in a subsequent step.

```powershell
param (
    [string]$ReleaseVariableName,
    [string]$ReleaseVariableValue      
)

$releaseurl = ('{0}{1}/_apis/release/releases/{2}?api-version=6.0' -f $($env:SYSTEM_TEAMFOUNDATIONSERVERURI), $($env:SYSTEM_TEAMPROJECTID),$($env:RELEASE_RELEASEID)  )

$release = Invoke-RestMethod -Uri $releaseurl -Method Get -Headers @{
    Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN"
}
$release.variables.($ReleaseVariableName).value="$ReleaseVariableValue"

$json = @($release) | ConvertTo-Json -Depth 10

Invoke-RestMethod -Uri $releaseurl -Method Put -Body $json -ContentType "application/json" -Headers @{Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN" }

```

To access a variable within the release stage itself, you can utilize the following script:

```powershell
Write-Host "##vso[task.setvariable variable=<VariableName>]<VariableValue>"
```

Replace `<VariableName>` with the name of the variable you want to set and `<VariableValue>` with the desired value for that variable.

## Running the Enhanced Script

1. Before running the script, ensure that you enable `Allow scripts to access the OAuth token` in Agent settings under additional options. This will allow the scripts to access the token.
2. Open PowerShell and navigate to the directory containing the script file.
3. Pass argument values with the release variable name and the actual values.

## Conclusion

By incorporating the script provided in this blog post, you can not only update release variables but also immediately utilize the updated values in subsequent steps, ensuring seamless communication between stages of your pipeline. This automation step is a valuable addition to any deployment workflow, enabling you to adapt to varying environments with ease. Happy automating!
