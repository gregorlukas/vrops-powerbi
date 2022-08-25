# High Level Architecture
![Take 3 - Reimagine vROps Reporting using - 3rd Party BI - PowerBI via REST High Level Architecture](https://user-images.githubusercontent.com/54750245/159884968-f71eb390-1723-4b29-8039-eef31bd88d8d.jpg)

# Prerequisites
* Windows Machine with access to vROps REST API and PowerBI 
* PowerBI Desktop https://powerbi.microsoft.com/en-us/desktop/ 
* Access to vROps via REST-API

# Setting up a Connection
1) Download the latest "vRealize Operations Manager via REST" Template (pbit) from: https://github.com/gregorlukas/vrops-powerbi/blob/main/pbit/vRealize%20Operations%20Manager%20PoC%20via%20REST.pbit
2) Open it with Microsoft PowerBI Desktop
3) Enter parameters when prompted (see documentation below) <br>
![PBIT Configuration Parameters](https://user-images.githubusercontent.com/54750245/186666097-c27cffbe-5b9b-4364-8d9e-ee73b0aef1a5.png)
4) Hit "Load" to load the data from vROps into PowerBI
5) Build your reports as desired

# Parameter Documentation
* **vROps Url**: 
<br>URL to vROps node with suite-api endpoint. This may be either a vROps Cluster VIP, vROps Nodeor Remote Collector. Parameter accepts any kind of address, such as FQDN, Short name or IP address. **NOTE**: Power BI enforces certificate checks. Please ensure that vROps Certificate is trusted on the machine running PowerBi Desktop either by installing a certificate signed by trusted CA or importing a self-signed certificate to the trusted certificate store. The address used for connection is included as Common Name (CN) or Subject Alternative Name (SAN) in the certificate.
* **vROps API Username**:
<br>The username to use for API authentication. Currently only local user are supported. The user needs at least read only REST-API access and access permission to objects that need to be visible in PowerBI. You can limit the amount of data visible in PowerBi by limiting the access permissions of this user. Required role permissions: Administration > REST APIs > Read Access to APIs
*  **vROps API Password**:
<br> Password for vROps API Username. **NOTE**: The password is visible in plain text when configuring and editing the data source.
*  **vROps Data Time Range**:
<br> This parameter controls the amount of data which will be loaded into PowerBI. It configures the time range relative to the current time and rollups of data. The parameter comes with six pre-configured combinations of time-ranges and rollups. Please use those as they will result in a reasonable amount of data. To make a custom configuration, select 9 - Custom and configure parameters labeled as (Custom Range) ... . 
<br>*NOTE:* Unless this parameter is set to `9 - Custom`, the following parameters stating with `(Custom Range)...` have no effect. 
<br>*WARNING:* Configuring large datasets (Large number of objects + long time range + granular data) may cause very long refresh times and harm vROps stability.
*  **vROps RollUp Type**:
<br> Unless `vROps Data Time Range` is set to `1 - Last Day, maximal granularity`, PowerBi will load aggregated data. This parameter controls which function to use for aggregation.
<br> It maps to the `rollUpType parameter of the `GET /api/resources/stats` REST-API endpoint of vROps. Please refer to vROps documentation for further information.
* **(Optional) Limit Scope by Parent ID**
<br>This optional parameter limits the scope of the data import to child object of one single parent. Most commonly this will be the object id of a Custom Group which contains all objects that should be visible in PowerBI. It takes the vROps object id of any object.
<br> ℹ️ *NOTE:* Only direct child objects are in scope. Descendants will not be included. E.g. providing an object ID of a vCenter Server will not include VMs, Hosts, Clusters, etc.
<br> ℹ️*NOTE:* Leave blank to load all objects into Power BI
* **(Optional) Limit Scope by Object Type**
<br> This optional parameter limits the dataset to specified object types. It takes a comma-separated list of ResourceKindKeys.
<br> ℹ️*NOTE:* Leave blank to load all objects into Power BI
* **(Optional) Limit Variety of Metrics**
<br>This optional parameter limits the metrics to a specified list. It take a comma-separated list of `statKeys`. 
<br>By default, PowerBI loads all available metrics for the objects in scope. As some objects have 600+ metrics, the amount of data may rise quickly. In order to limit the number of metrics collected per object,
<br> ℹ️ NOTE: The amount of metrics does not impact loading and refresh times negatively. Loading and refresh time scales with the number of objects in scope.
<br> ℹ️ NOTE: This parameter takes comma-separated statKeys only. No wildcards or metric-names supported.


# Data Model in PowerBI
Power BI pulls data from vROPs via REST API calls to create a data model representing the following structures of vROps as tables in PowerBI:
* Objects
* Alerts
* Metrics
* Properties

The following tables shows an overview of data tables available in PowerBI as a result of this integration:
| Table | Associated Query / REST Call | Description |
|:--|:--|:--|
|**vROps - Alert Definitions**|/suite-api/api/alertdefinitions|All Alert Definitions in the System. Used to lookup metadata for active alerts|
|**vROps - Alerts**|/suite-api/api/alerts|Alerts (active and historical) for all objects in scope. Queried once per refresh.|
|**vROps - Metric Metadata**|/suite-api/api/adapterkinds/<AdapterKindKey>/resourcekinds/<ResourceKindKey>/statkeys|Metric metadata such as human readable names and descriptions. Used to look up metadata for metrics and properties. Queried per resource kind|
|**vROps - Metrics (Tall)**|/suite-api/api/resources/stats/query|Time series metric data for all objects in scope in tall data format. Queried per resource. May be used to as source for custom pivot tables.|
|**vROps - Properties (Tall)**|/suite-api/api/resources/properties|Latest property data for all objects in scope in tall data format.|
|**vROps - Resources**|/suite-api/api/resources|List of Objects in scope incl. some metadata. One query per refresh. Basis for most other queries.|
|**vROps - Resource Identifier**|/suite-api/api/resources|All object identifiers in tall data format. One query per refresh.|
|**vROps - Resource Relations**|/suite-api/api/resources|Relations of objects in parent / child tuples. One query per refresh.|
|**vROps URL**|Parameter|Shows to which vROps URL PowerBI is connected.|

# Limitations
* Authentication via vROps Local Users only
* Import limited to 10.000 resources (max page size of /api/resources
* Large queries may impact vROps performance
* All dates in UTC

