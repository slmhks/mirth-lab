# Interface Inventory

|Channel|Source|Transformation|Destination|Protocol|Purpose|
|-|-|-|-|-|-|
|CH01\_IN\_ADT\_MLLP|TCP Listener :6661|Validate and map ADT|File Writer / response|MLLP|Process ADT A01/A08|
|CH02\_IN\_SIU\_MLLP|TCP Listener :6662|Filter SIU events|File Writer / response|MLLP|Route SIU S12/S15|
|CH03\_OUT\_ORM\_MLLP|File Reader|CSV to ORM^O01|SmartHL7 :7771|MLLP|Send radiology orders|
|CH04\_OUT\_ORU\_MLLP|File Reader|XML to ORU^R01|SmartHL7 :7772|MLLP|Send laboratory results|
|CH04\_OUT\_ORU\_MLLP\_RECOVERY|File Reader|XML to ORU^R01|Test receiver :7774|MLLP|Queue and recovery testing|
|CH\_TEST\_FAKE\_RIS\_AE|TCP Listener :7774|Build controlled AE ACK|Source response / File Writer|MLLP|Simulate application rejection|
|CH05\_ORU\_FILE\_TO\_FHIR\_R4|File Reader|ORU to FHIR transaction Bundle|HAPI FHIR R4|HTTPS|Submit FHIR resources|



## Environment

* Mirth Connect: 4.5.2
* PostgreSQL: 17
* Container runtime: Docker Desktop
* Host operating system: Windows 11
* FHIR version: R4
* HL7 version: 2.3
* Test data: Synthetic only

