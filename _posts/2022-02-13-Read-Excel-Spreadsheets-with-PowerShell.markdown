---
layout: single
classes: wide
share: false
title: Read Excel Spreadsheets with PowerShell
tags: [PowerShell]
---
My team owns a system that accepts input as an Excel spreadsheet, and sometimes we need to automate tasks given that input.  The [Open XML SDK](https://docs.microsoft.com/en-us/office/open-xml/open-xml-sdk) seemed like the obvious choice but it would require assemblies to be loaded.

Looking into the Open XML SDK it started to look like just an XML parser with way more capabilities than I needed.  It also seemed possible to just read the Excel spreadsheet using XML and XPath, and that turned out to be true.

The scenario I have is very simple, read text from cells in the first worksheet.  For more elaborate or complex scenarios I would look at the Open XML SDK again but for now I am able to accomplish this scenario without the SDK.

The spreadsheet in the example has data that looks like this:

| |A  | B |
--- | --- | ---|
|1|Cell A1|Cell B1|
|2|Cell A2|Cell B2|

The sample below shows how to read cell data from a spreadsheet using just PowerShell.

```powershell
### Copy the XL to a ZIP
Copy-Item -Path MySpreadsheet.xlsx -Destination MySpreadsheet.xlsx.zip

### Get the files from the ZIP
Expand-Archive -Path MySpreadsheet.xlsx.zip -DestinationPath MySpreadsheet.xlsx.zip.expanded -Verbose
<#
VERBOSE: Performing the operation "Create Directory" on target "Destination: C:\Foo\MySpreadsheet.xlsx.zip.expanded".
VERBOSE: Preparing to expand...
VERBOSE: Performing the operation "Expand-Archive" on target "C:\Foo\MySpreadsheet.xlsx.zip".
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\[Content_Types].xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\_rels\.rels'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\xl\workbook.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\xl\_rels\workbook.xml.rels'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\xl\worksheets\sheet1.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\xl\theme\theme1.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\xl\styles.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\xl\sharedStrings.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\docMetadata\LabelInfo.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\docProps\core.xml'.
VERBOSE: Created 'C:\Foo\MySpreadsheet.xlsx.zip.expanded\docProps\app.xml'.
#>

### Use XPath to get the Shared Strings
$sharedStrings = Select-Xml -Path MySpreadsheet.xlsx.zip.expanded\xl\sharedStrings.xml -XPath '/main:sst/main:si/main:t' -Namespace @{'main' ="http://schemas.openxmlformats.org/spreadsheetml/2006/main"}
$sharedStrings.Node.'#text'
<#
Cell A1
Cell B1
Cell A2
Cell B2
#>

### Use XPath to get Sheet1
$cells = Select-Xml -Path MySpreadsheet.xlsx.zip.expanded\xl\worksheets\sheet1.xml -XPath '/main:worksheet/main:sheetData/main:row/main:c[main:v]'-Namespace @{'main' ="http://schemas.openxmlformats.org/spreadsheetml/2006/main"}

### Create a Hashtable of Cell Name and Cell Value
$cellValues = @{}
for ($c = 0; $c -lt $cells.Count; $c++)
{ 
    $cellValues.Add($cells[$c].Node.r, $sst.Node[$cells[$c].Node.v].'#text')
}

$cellValues 
<#
Name Value  
---- -----  
A1   Cell A1
B2   Cell B2
A2   Cell A2
B1   Cell B1
#>

```
 

