---
layout: single
title: Get all Running Runbook Jobs
date: '2012-06-02T04:27:00.001-07:00'
modified_time: '2012-06-06T11:41:13.080-07:00'
tags:
- edm.guid
- scorch
- System Center Orchestrator 2012
- xml  More stats
- guid
- filter runbooks
- PowerShell
- Orchestrator
- GET Request
- Orchestrator Web Service
- orchestrator.svc
- odata
blogger_id: tag:blogger.com,1999:blog-603465820850819373.post-3174606743280457125
blogger_orig_url: http://jmattivi.blogspot.com/2012/06/get-all-running-runbook-jobs.html
---

This post I'll show how to get all Running Runbook jobs.  
  
I used the ConvertTo-SCOObject function Robert Hearn posted here to parse the xml response from the GET call.  
  
```
  
function ConvertTo-SCOObject {  
\[CmdletBinding()\]  
PARAM (  
\[Parameter(Mandatory=$true)\]  
\[String\] $XmlFeed  
)  
$xml =\[xml\]$XmlFeed  
  
$ns = New-Object Xml.XmlNamespaceManager $xml.NameTable  
$ns.AddNamespace( "d", "http://schemas.microsoft.com/ado/2007/08/dataservices")  
$ns.AddNamespace( "m", "http://schemas.microsoft.com/ado/2007/08/dataservices/metadata")  
$ns.AddNamespace( "def", "http://www.w3.org/2005/Atom")  
  
$objarray = @()  
  
$entryNodes = $xml.SelectNodes("//def:entry",$ns)  
foreach ($entryNode in $entryNodes)  
{  
$obj = New-Object PSObject  
$objectType = $entryNode.SelectSingleNode(".//def:category", $ns).GetAttribute("term")  
  
$obj | add-member Noteproperty -Name "Object Type" -Value $objectType.Split(".")\[-1\]  
  
$propertiesNode = $entryNode.SelectSingleNode(".//m:properties", $ns)  
foreach($propertyNode in $propertiesNode.ChildNodes)  
{  
$obj | add-member Noteproperty -Name $propertyNode.LocalName -Value $propertyNode.get_InnerText()  
}  
  
$links = $entryNode.SelectNodes(".//def:link", $ns)  
foreach ($link in $links)  
{  
$obj | add-member NoteProperty -Name $link.title -Value $link.href  
}  
$objarray += $obj  
}  
  
return $objarray  
  
```  
  
Due to the paging limits on the web service collections (Jobs being 500), I used this method for filtering the results.  The most important part of this GET is building the query into the odata URI.  You can build in filters, select statements, etc. on the Web Service collections shown like this.  
  
http://scorch.domain:81/Orchestrator2012/Orchestrator.svc/Jobs()?$filter=Status eq 'Running'  
\\_______________________________________________________/ \\____/  \\_________________________/  
                           |                                |                   |  
                    service root URI                  resource path        query options  
  
Putting this into practice w/ SCOrch, you can query the Jobs collection, filter for Jobs that have a Status of "Running", and select the fields you would like to return – Id,RunbookId,LastModifiedTime,Status.  
  
"http://scorch.domain:81/Orchestrator2012/Orchestrator.svc/Jobs()?`$filter=Status eq 'Running'&`$select=Id,RunbookId,LastModifiedTime,Status"  
  
So you can use this on any of the collections to filter results i.e. Jobs, Runbooks, RunbookInstances, Statistics, etc.  
  
The GET request would look like this.  
  
$creds = Get-Credential("domain\\username")  
  
$XmlFeed = $null  
$count = 0  
$url = "http://scorch.domain:81/Orchestrator2012/Orchestrator.svc/Jobs()?`$filter=Status eq 'Running'&`$select=Id,RunbookId,LastModifiedTime,Status"  
$request = \[System.Net.HttpWebRequest\]::Create($url)  
$request.Credentials = $creds  
$request.Timeout = 60000  
$request.ContentType = "application/atom+xml,application/xml"  
$request.Method = "GET"  
  
$response = $request.GetResponse()  
$requestStream = $response.GetResponseStream()  
$readStream=new-object System.IO.StreamReader $requestStream  
$XmlFeed = $readStream.ReadToEnd()  
$readStream.Close()  
$response.Close()  
  
Using the ConvertTo-SCOObject function, you can then parse the response and set your variables and get a count of the running runbooks.  
  
ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$count++}  
$Id = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.Id}  
$RunbookId = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.RunbookId}  
$LastModifiedTime = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.LastModifiedTime}  
$Status = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.Status}  
  
Here is the complete script for getting all Running Runbook Jobs.  Highlighted fields would need to be updated per your environment.  
  
```
function ConvertTo-SCOObject {  
\[CmdletBinding()\]  
PARAM (  
\[Parameter(Mandatory=$true)\]  
\[String\] $XmlFeed  
)  
$xml =\[xml\]$XmlFeed  
  
$ns = New-Object Xml.XmlNamespaceManager $xml.NameTable  
$ns.AddNamespace( "d", "http://schemas.microsoft.com/ado/2007/08/dataservices")  
$ns.AddNamespace( "m", "http://schemas.microsoft.com/ado/2007/08/dataservices/metadata")  
$ns.AddNamespace( "def", "http://www.w3.org/2005/Atom")  
  
$objarray = @()  
  
$entryNodes = $xml.SelectNodes("//def:entry",$ns)  
foreach ($entryNode in $entryNodes)  
{  
$obj = New-Object PSObject  
$objectType = $entryNode.SelectSingleNode(".//def:category", $ns).GetAttribute("term")  
  
$obj | add-member Noteproperty -Name "Object Type" -Value $objectType.Split(".")\[-1\]  
  
$propertiesNode = $entryNode.SelectSingleNode(".//m:properties", $ns)  
foreach($propertyNode in $propertiesNode.ChildNodes)  
{  
$obj | add-member Noteproperty -Name $propertyNode.LocalName -Value $propertyNode.get_InnerText()  
}  
  
$links = $entryNode.SelectNodes(".//def:link", $ns)  
foreach ($link in $links)  
{  
$obj | add-member NoteProperty -Name $link.title -Value $link.href  
}  
$objarray += $obj  
}  
  
return $objarray  
}  
  
$creds = Get-Credential("domain\\username")  
  
$XmlFeed = $null  
$count = 0  
$url = "http://scorch.domain:81/Orchestrator2012/Orchestrator.svc/Jobs()?`$filter=Status eq 'Running'&`$select=Id,RunbookId,LastModifiedTime,Status"  
$request = \[System.Net.HttpWebRequest\]::Create($url)  
$request.Credentials = $creds  
$request.Timeout = 60000  
$request.ContentType = "application/atom+xml,application/xml"  
$request.Method = "GET"  
  
$response = $request.GetResponse()  
$requestStream = $response.GetResponseStream()  
$readStream=new-object System.IO.StreamReader $requestStream  
$XmlFeed = $readStream.ReadToEnd()  
$readStream.Close()  
$response.Close()  
  
ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$count++}  
$Id = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.Id}  
$RunbookId = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.RunbookId}  
$LastModifiedTime = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.LastModifiedTime}  
$Status = ConvertTo-SCOObject -XmlFeed $XmlFeed | %{$_.Status}  
```
  
A few good resources for working with the Orchestrator Web Service.  
  
[http://blogs.technet.com/b/scorch/archive/2011/06/17/fun-with-the-orchestrator-2012-beta-the-web-service-and-powershell.aspx](http://blogs.technet.com/b/scorch/archive/2011/06/17/fun-with-the-orchestrator-2012-beta-the-web-service-and-powershell.aspx)  
  
[http://www.odata.org/developers/protocols/uri-conventions](http://www.odata.org/developers/protocols/uri-conventions)