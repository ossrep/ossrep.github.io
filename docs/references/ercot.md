# ERCOT Integration

ERCOT has some public endpoints to get access to reports. 

The main report list is available at [https://www.ercot.com/misapp/servlets/IceMktRepListWS](https://www.ercot.com/misapp/servlets/IceMktRepListWS)  

Below is a snippet of the return XML structure. 

```xml
<ns0:ListReportTypeSchemaRes xmlns:ns0="http://www.ercot.com/schema/2012-10/nodal/ListReportTypeSchemaRes">
    <ns0:ReportTypeList>
        <ns0:Report>
            <ns0:ReportTypeID>12340</ns0:ReportTypeID>
            <ns0:ReportName>System-Wide Demand</ns0:ReportName>
            <ns0:SecurityClassification>PUBLIC</ns0:SecurityClassification>
        </ns0:Report>
        <ns0:Report>
            <ns0:ReportTypeID>203</ns0:ReportTypeID>
            <ns0:ReportName>TDSP ESI ID EXTRACT</ns0:ReportName>
            <ns0:SecurityClassification>PUBLIC</ns0:SecurityClassification>
        </ns0:Report>
    </ns0:ReportTypeList>
</ns0:ListReportTypeSchemaRes>
```

For each report in the report list, there is a ReportTypeID. This can then be used to query for all the available reports based on that ReportTypeID. 


[https://www.ercot.com/misapp/servlets/IceDocListJsonWS?reportTypeId=203](https://www.ercot.com/misapp/servlets/IceDocListJsonWS?reportTypeId=203)

Below is a snippet of the return JSON structure. 

```json
{
    "ListDocsByRptTypeRes": {
        "DocumentList": [
            {
                "Document": {
                    "ExpiredDate": "2026-07-04T23:59:59-05:00",
                    "ILMStatus": "EXT",
                    "SecurityStatus": "P",
                    "ContentSize": "15014",
                    "Extension": "zip",
                    "ReportTypeID": "203",
                    "Prefix": "ext",
                    "FriendlyName": "LUBBOCK______DAILY",
                    "ConstructedName": "ext.00000203.0000000000000000.20260603.055219393.LUBBOCK______DAILY.zip",
                    "DocID": "1234254362",
                    "PublishDate": "2026-06-03T05:52:19-05:00",
                    "ReportName": "TDSP ESI ID EXTRACT",
                    "DUNS": "0000000000000000",
                    "DocCount": "0"
                }
            },
            {
                "Document": {
                    "ExpiredDate": "2026-07-04T23:59:59-05:00",
                    "ILMStatus": "EXT",
                    "SecurityStatus": "P",
                    "ContentSize": "696562",
                    "Extension": "zip",
                    "ReportTypeID": "203",
                    "Prefix": "ext",
                    "FriendlyName": "CENTERPOINT__DAILY",
                    "ConstructedName": "ext.00000203.0000000000000000.20260603.055219302.CENTERPOINT__DAILY.zip",
                    "DocID": "1234254361",
                    "PublishDate": "2026-06-03T05:52:19-05:00",
                    "ReportName": "TDSP ESI ID EXTRACT",
                    "DUNS": "0000000000000000",
                    "DocCount": "0"
                }
            },
            {
                "Document": {
                    "ExpiredDate": "2026-07-04T23:59:59-05:00",
                    "ILMStatus": "EXT",
                    "SecurityStatus": "P",
                    "ContentSize": "1098000",
                    "Extension": "zip",
                    "ReportTypeID": "203",
                    "Prefix": "ext",
                    "FriendlyName": "ONCOR_ELEC___DAILY",
                    "ConstructedName": "ext.00000203.0000000000000000.20260603.055219164.ONCOR_ELEC___DAILY.zip",
                    "DocID": "1234254360",
                    "PublishDate": "2026-06-03T05:52:19-05:00",
                    "ReportName": "TDSP ESI ID EXTRACT",
                    "DUNS": "0000000000000000",
                    "DocCount": "0"
                }
            }
        ]
    }
}
```

Then for all the Document entries in this DocumentList, there is a `DocID` value which can then be used to download the actual document. 

[https://www.ercot.com/misdownload/servlets/mirDownload?doclookupId=1234254360](https://www.ercot.com/misdownload/servlets/mirDownload?doclookupId=1234254360)

This URL will download the file using the Document's `ConstructedName` as the default name and in the format described by `Extension`.

From there, each report should have it's own logic to unzip if necessary and parse the report-specific data structures. 
