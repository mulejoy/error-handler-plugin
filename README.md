# Error Handler Plugin

Error Handling Plugin is a framework that facilitate the handling of errors produced by any Mule application. This includes both technical and business errors. 

The objective of the Error Handling Plugin is to centralize the error handling mechanism and to provide operation units facility to identify and configure the error handling rule. The current framework incorporates replaying, email notification and support ticket creation features.

# Working Principle

Whenever an error occurs, the application captures the error and invokes a handler flow from the error handler plugin. 

Based on a set of configurable information passed by the application and the choice of a messaging service selected by the developer (currently supports RabbitMQ and Apache Kafka), the error is first sent to error-orchestrator and then further processed to designated error handling mechanisms.

* The application's exception handling should **Collect** the original error context and send it to a queue for errors (queue_errors).
* The error orchestrator **Listens** for messages from queue_errors and then, based on the options configured in payload by the developer, send the error message to queue_errors_replay, queue_errors_email, and/or queue_errors_issue_ticket.
* Each queue have a service built to **Listen** for messages from the corresponding queue and perform an action that is intended to be performed.

## How to Publish the Plugin?

Configure the connector dependency as follows:

_ORG_ID_ must be replaced with your respective Organization ID

```
<groupId>ORG_ID</groupId>
<artifactId>error-handler-plugin</artifactId>
<version>1.0.0</version>
<classifier>mule-plugin</classifier>
```

Ensure that the distributionManagement property in pom.xml is configured with the exchange repository pointing to your organization
```
<distributionManagement>
    <!-- Target Anypoint Organization Repository -->
    <repository>
        <id>Exchange2</id>
        <name>Exchange2 Repository</name>
        <url>https://maven.anypoint.mulesoft.com/api/v1/organizations/${project.groupId}/maven</url>
        <layout>default</layout>
    </repository>
</distributionManagement>
```

Once the above-mentioned configurations are enabled in your pom.xml, execution of maven deploy goal _**'mvn deploy'**_ 
should be able to publish the connector asset to your organization.

**IMPORTANT:** You have to manually delete the previous asset from your exchange within 7 days of deployment since you can't deploy the same version to Exchange, or increase the version in the pom.xml to publish a newer version of the asset.

To test the plugin in your local enviornment, comment the below section in pom.xml. This is added to avoid packaging the properties file as part of the error handler plugin.
```
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <excludes>
            <exclude>*.properties</exclude>
        </excludes>
        <filtering>false</filtering>
    </resource>
</resources>
```

Create **config.properties** under _/src/main/resources_ and set the necessary keys to configure RabbitMQ and Apache Kafka. <br>

Refer the below section for the keys to be configured in properties.

## How To Configure the Plugin

Configure the Connector dependency as below in your application pom.xml:

_ORG_ID_ must be replaced with your respective Organization ID
```
<groupId>ORG_ID</groupId>
<artifactId>error-handler-plugin</artifactId>
<version>1.0.0</version>
<classifier>mule-plugin</classifier>
```
---
Below properties have to configured in the application, depending on which messaging service is used in the customer's infrastructure
```
# Kafka Configuration
kafka.bootstrap.server=xx.xx.xx.xx:9092

# Rabbit MQ Configuration
rabbitMQ.host=xx.yy.cloudamqp.com
rabbitMQ.port=5672
rabbitMQ.vHost=virtualhost
rabbitMQ.uname=virtualhost
rabbitMQ.pwd=password

# Email Configuration
email.host=smtp.gmail.com
email.port=587
email.user=xx.abc@gmail.com
email.pwd=password
```
**Note**: **rabbitMQ.pwd** and **email.pwd** will have to be created as Secure Properties. Hence, configuring the same as non-secure properties would fail deployment.

To **Import** the flows from error-handler-plugin:

Follow the standard procedure to Create a 'Import' configuration from global elements and define the XML that is to be imported.
* For RabbitMQ - error-handler-rabbitMQ.xml
* For Apache Kafka - error-handler-kafka.xml

**Example**
```
<import doc:name="Import" doc:id="52f0ab58-151d-4b13-b739-b57ca741ccb4" file="error-handler-rabbitMQ.xml" />
<import doc:name="Import" doc:id="1ff58d3f-325c-43dc-8aea-da1b7ecc3be4" file="error-handler-kafka.xml" />
```
Under Error Handling at Processor level / Flow level / Application level, a standard payload has to be created and set in variable : '**error**' before invoking the flows to push the message to queue_errors.

The error handler plugin will access the _#[vars.error]_ object to further process it
```
%dw 2.0
output application/java
---
{
	errorObj : error,   
	customErrorMsg : null, 
	errorReplay :  false, 
	errorEmail : {
		sub : null,
		to : "xx.abc@gmail.com",
		cc : null,
		body: null
	},
	errorIssueTicket : false,
	appDetails: {
		appName : app.name,
		flowName : flow.name,
		env : vars.env default "DEV"
	}
}
   
```

_errorObj_ - Mandatory, and should not be changed <br>
_customErrorMsg_ - Optional, if you want customized error message to be logged <br>
_errorReplay_ - Optional, and defaulted to false. Upcoming Feature <br> 
_errorEmail_ - Optional, first create an object with details required to send an email. <br>
* sub - Subject of email (Optional: Defaulted to [env] : [appname] - Exception Occurred) <br>
* to - Recipient List (Mandatory) <br>
* cc - Carbon Copy Recipients (Optional: Defaulted to 'to' list) <br>
* body - Message body of Email (Optional: Defaulted to 'Please contact Support Team for further details') <br>

_errorIssueTicket_ - Optional, defaulted to false (upcoming feature) <br>
_appDetails_ - (Mandatory)
* appName - Application Name
* flowName - Flow Name
* env - Environment

Once the payload is created, use a flow Reference to invoke:
* For RabbitMQ - handleError-rabbitMQ
* For Apache Kafka - handleError-apacheKafka 

## 1.0.0 version - Release notes

**Features:**
* Reduces development effort and hours, which eventually saves money.
  Separates error handling logic from the application, making maintenance simpler.
* Enhances the agility of the team to bring new products and features as the team does not need to consider error handling.
* Gives better visibility as to why an error occurred.
* Has the ability to send an email with the error details
* It’s very easy to install, easy to manage and easy to customize.

**Upcoming Features:**
* Enables replay of an asynchronous process (ETL, batch etc) by itself. It can be used to resolve intermittent network errors.
* Has the ability to create an issue ticket by itself and assign it to a group.

## Author

## Support disclaimer
It won't be officially supported by MuleSoft as it is considered a custom connector. 
