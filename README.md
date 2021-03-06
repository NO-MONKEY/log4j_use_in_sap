# log4j_use_in_sap
Some findings about where Log4J might be in use in SAP products

While SAP provides KBA's and down the line patches for Log4Shell vulnerabilities in their products and flags them with the CVE CVE-2021-44228 (You can search for this with access to the support launchpad via https://launchpad.support.sap.com/#/solutions/notesv2/?q=CVE-2021-44228&sortBy=score&sortOrder=desc) security researchers and SAP customers search for ways to mitigate the risk.
Therefore it's important to prioritize where to search for this vulnerability. 

## Further source of information:
- [Q&A feed on answers.sap.com](https://answers.sap.com/questions/13547385/sap-and-log4j.html)
- [Official SAP Trust Office statement (supposed to be updated)](https://support.sap.com/content/dam/support/en_us/library/ssp/my-support/trust-center/sap-tc-01-5025.pdf)
- [Blog of how to determine and fix log4shell in HANA XS](https://blogs.sap.com/2021/12/14/hana-xsa-log4j-cve-2021-44228/) 
- The SAP RFC connection library [JcO](https://support.sap.com/en/product/connectors/jco.html) uses Log4j in the class __Log4jTraceWriter__. Due to the package path __org.apachge.log4j__ it seems that still Log4J in version 1 is being used as with version 2 the path changed to __org.apache.*logging*.log4j__. Please consider that the software using JcO might use another version of log4j too. See details from the decompilation below:

```java
import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.apache.log4j.Priority;

public final class Log4jTraceWriter
extends Trace
implements JCoTraceWriter {
    private static Log4jTraceWriter self = null;
    private Logger logger = Logger.getLogger((String)"com.sap.conn.jco");
...
@Override
    public void trace(int level, String message) {
        try {
            if (level == 0) {
                this.logger.fatal((Object)message);
            } else if ((level & 1) != 0) {
                this.logger.error((Object)message);
            } else if ((level & 2) != 0) {
                this.logger.warn((Object)message);
            } else if ((level & 4) != 0) {
                this.logger.info((Object)message);
            } else if ((level & 8) != 0) {
                this.logger.debug((Object)message);
            } else if ((level & 0x10) != 0 || (level & 0x20) != 0) {
                this.logger.trace((Object)message);
            } else {
                this.logger.log((Priority)Level.ALL, (Object)message);
            }
        }
        catch (Throwable throwable) {
            // empty catch block
        }
    }
```
On a \*NIX like system you should be able to check for the Log4J lib(s) loaded to memory with:
```
sudo ls -l /proc/*/fd/ | grep log4
```

## Parsing the SAP product documentation

We've been using the official SAP documentation ([help.sap.com](https://help.sap.com)] as a source of for a web research.

By using the elastic search query 'https://help.sap.com/http.svc/elasticsearch?area=content&deliverable=&version=&language=en-US&state=PRODUCTION&q=%27log4j&transtype=standard,html,pdf,others&product=&to=5000&advancedSearch=0&excludeNotSearchable=1' we retriefed the first 5k matches in help.sap.com and parsed the json result about the products with exact matches for __log4j__, __Log4j__, and __Log4J__. 

## Disclaimer
This is by no means a proof that the product is vulnerable for Log4Shell! This is meant to be a help to prioritise  investigation.

The filtered output including the elastic search finding details with the exact reference URL are available in json format [here](https://github.com/NO-MONKEY/log4j_use_in_sap/blob/main/log4j_sap_products.json).

This compiles into a list of the following unique SAP product names:

SAP Commerce
- SAP Omnichannel Banking
- SAP Mobile Platform Server 3.1
- SAP Work Manager
- SAP Adaptive Server Enterprise
- SAP BusinessObjects Business Intelligence Platform
- SAP HANA Smart Data Integration and SAP HANA Smart Data Quality
- SAP Business Technology Platform
- SAP Commerce Cloud, Integration Extension Pack
- SAP Event Stream Processor
- SAP IQ
- SAP IoT services for SAP BTP for the Cloud Foundry Environment
- SAP Underwriting for Insurance
- SAP HANA Platform
- SAP Enterprise Threat Detection
- SAP SuccessFactors Learning
- SAP Information platform services
- SAP Data Services
- SAP Ariba cloud integration
- SAP Big Data Services
- SAP HANA Streaming Analytics
- SAP IoT Application Enablement
- SAP PowerDesigner
- SAP Logistics Business Network, Freight Collaboration Option
- SAP EHS Regulatory Documentation OnDemand
- SAP Product Lifecycle Management for Digital Products
- SAP Enterprise Threat Detection Log Collector
- SAP Quotation and Underwriting for Insurance
- SAP Process Integration
- SAP Product Lifecycle Management for Insurance
- SAP HANA Hadoop Integration
- SAP Data Hub
- SAP Edge Services, cloud edition
- SAP Customer Activity Repository applications bundle
- SAP Predictive Analytics
- SAP Application Logging Service for SAP BTP
- SAP Information Steward
- SAP Mobile Consumer Assistant by GK
- SAP Point-of-Sale
- SAP Regulation Management 1.0, AVM Mitigation Control Functional
- SAP HANA Cloud, SAP HANA Database
- SAP BusinessObjects Explorer
- SAP BTP SDK for Android
- SAP Mobile Platform Server 3.0
- SAP Mobile Services
- SAP Inventory Manager
- SAP Mobile Platform - Mobiliser
- BusinessObjects Knowledge Accelerator, version for BusinessObjects Voyager
- SAP Enterprise Point-of-Sale
- SAP Fortify by Micro Focus
- SAP Edge Services, on-premise edition
- SAP Commerce Cloud in the Public Cloud
- SAP Sourcing and SAP Contract Lifecycle Management
- SAP Business One analytics powered by SAP HANA
- SAP Convergent Mediation by DigitalRoute
- SAP LoadRunner by Micro Focus
- SAP Roambi
- SAP Quality Center by Micro Focus
- SAP Customer Relationship Management
- SAP Business One, version for SAP HANA
- SAP API Management On-Premise
- SAP Cloud Platform
- SAP Enable Now
