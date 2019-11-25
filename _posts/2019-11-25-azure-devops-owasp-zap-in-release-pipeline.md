---
title: 'Azure DevOps : OWASP ZAP In Release Pipeline'
---

Seting up OWASP ZAP In Azure devops release pipeline for API & UI

`$(System.DefaultWorkingDirectory)`

```
chmod -R 777  ./

docker run --rm -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t https://example.com -g gen.conf -x OWASP-ZAP-Report.xml -r scan-report.html

true
```


```
$XslPath = "$($Env:SYSTEM_DEFAULTWORKINGDIRECTORY)/_QualityAssurance/SecurityTest/OWASPToNUnit3.xslt"
$XmlInputPath = "$($Env:SYSTEM_DEFAULTWORKINGDIRECTORY)/OWASP-ZAP-Report.xml"
$XmlOutputPath = "$($Env:SYSTEM_DEFAULTWORKINGDIRECTORY)/Converted-OWASP-ZAP-Report.xml"
$XslTransform = New-Object System.Xml.Xsl.XslCompiledTransform
$XslTransform.Load($XslPath)
$XslTransform.Transform($XmlInputPath, $XmlOutputPath)
```

`$(System.DefaultWorkingDirectory)`


[Link](https://gist.github.com/sudhinsr/6dad07c20785d8d00ffd406a6c581b15)


*Resources:*

* ZAP API Scan : [https://github.com/zaproxy/zaproxy/wiki/ZAP-API-Scan](https://github.com/zaproxy/zaproxy/wiki/ZAP-API-Scan)
* ZAP Full Scan : [https://github.com/zaproxy/zaproxy/wiki/ZAP-Full-Scan](https://github.com/zaproxy/zaproxy/wiki/ZAP-Full-Scan)
* ZAP In Azure [https://devblogs.microsoft.com/premier-developer/azure-devops-pipelines-leveraging-owasp-zap-in-the-release-pipeline](https://devblogs.microsoft.com/premier-developer/azure-devops-pipelines-leveraging-owasp-zap-in-the-release-pipeline)
