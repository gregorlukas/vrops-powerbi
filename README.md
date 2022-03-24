# Sample VMware vRealize Operations to Microsoft PowerBI Integration
This demo provides a PowerBI Templates (pbit files) to load data from VMware vRealize Operations into Microsoft PowerBI for reporting purposes. 

## Prerequisites
- PowerBI Desktop in a Windows Machine
- Office 360 subscription 
- Read Access to vROps REST-API 
- Internet Access for Windows Machine.

## Integration Architecture

### Basic Proof of Concept Architecture 
![image2021-5-28_14-10-5](https://user-images.githubusercontent.com/54750245/159876927-f0b21c99-e2cf-46e4-8815-aef52fff8763.png)

### Advanced Architecture for Automated Data Refresh
![image2021-5-28_14-10-5](https://user-images.githubusercontent.com/54750245/159877001-57b66f98-f5f3-4214-b546-6b7e65de4d68.png)

## User Guides
For simple, PoC style implementations see [How to Get vRealize Operations Data into Microsoft PowerBI](https://github.com/gregorlukas/vrops-powerbi/blob/main/How%20to%20Get%20vROps%20Data%20into%20PowerBI.md).

For production grade implementations with scheduled, incremental refreshes see How to Schedule Incremental Data Refresh of vROps Data in PowerBI .

## Known Issues
### Known Issue 1
#### Issue
Loading data fails with:
Formula.Firewall: Query xxx references other queries or steps, so it may not directly access a data source. Please rebuild this data combination.
#### Description
Due to the nature of vROps REST API Authentication, all queries depend on the "vROps Token" query. Such interdependencies are prevented by default in PowerBI and need to be ignored in settings.
#### Workaround
Ignore PowerBI Privacy Levels

![image](https://user-images.githubusercontent.com/54750245/159880025-4987864a-a4c6-4844-a689-7355c94956d1.png)

### Known Issue 2
#### Issue
Loading data fails with authentication errors.
#### Description
Sometimes the query-order does not refresh the API token and causes all other queries to fail
#### Workaround
Manually refresh "Helper/vROps Token" in query editor.

### Known Issue 3
#### Issue
SSL / TLS Errors similar to "The underlying connection was closed: Cloud not establish trust relationshipü for the SSL/TLS secure channel."
#### Description
Power BI is not able to verify SSL certificate used by vROps. 
#### Workaround
Power BI enforces certificate checks. Please ensure that:

vROps Certificate is trusted on the machine running PowerBi Desktop
either by installing a certificate signed by trusted CA
or importing a self-signed certificate to the trusted certificate store
The address used for connection is included as Common Name (CN) or Subject Alternative Name (SAN) in the certificate

 
