# Introduction
Check out various SAP Integration Suite scenarios with different adapters and pallete functions! This bundle includes:
- Data fetching from API.
- Error handling using Exception Subprocess.
- Break down complex iFlows into smaller chunks using Local Integration Process.
- Groovy Script operations.
- Sending email using MAIL adapters.
- JDBC adapter operations.
- SOAP adapter operations.
- SFTP adapter operations.

Download the entire package and import in your Integration Suite from [iflow_scenerios.zip](/iflow_scenerios.zip).

Or download individual iFlows from [Individual iFlows](/individual_iFlows).

# Background
I have been using SAP Integration Suite professioinaly since last year. I have encountered lots of bottlenecks including but not limiting to configuration and the way the message flows in iFlows. Through my errors I learned a lot and still learning. I created this bundle as a reference for aspirants who just started to learn SAP CPI and need some real world use case scenarios to practice. I won't include step by step guide for each iFlows, but basic configuration and highlight where you might run into an error. Treat this as a reference for your practice rather than a guide book. With all that said let's move on with the scenarios!

# Prerequisites
Before we start make sure you meet all the prerequisites to successfully deploy all the iFlows
- Access to SAP BTP Integration Suite.
- All required roles and Capabilities added.
- Rapid API account and subscription to APIs (OpenWeather, CurrencyConverter, CricBuzz).
- Credentials for MAIL and SFTP. 
- SAP HANA setup and JDBC data source setup.
- Postman for testing your iFlows.

# iFlows
Now, time to implement the iFlows. I would include the scenarios for every iFlow, some heads-up to errors you might encounter and a brief summary of what the iFlow actually does.

### 1. Cricbuzz Data Aggregation (Match Results and Players) [Download](/individual_iFlows/_1_Cricbuzz_Data_Aggregator.zip).
You want to check a specific teams scorecard. You pick a match which interests you and want to check it's result and which players played on that match. ALL combined in a single payload, Sent to your Mail!

1. Expose an HTTPS adapter and use the endpoint for Postman testing.
2. Use a content modifier to set header for your rapid api host and key.
3. Use Request-Reply to fetch teamId data from CricBuzz (teams/list).
4. Use teamId as Key and call Second API teams/get-results.
5. Use teamId and matchId as Key and call third API matches/get-team to fetch the List of Players playing
that match.
6. Combine the payload of payload of Second and Third API and send to your Mail
7. Add Exception Subprocess for error handling (Best Practice).

Make sure to convert the response JSON data from APIs to XML for processing in and keep in mind the json data from apis contain '_1h' type of values. They dont't get converted to XML, you will need a groovy script to modify these values into acceptable xml format.

### 2. Current Weather and 5 day forecast using Local Integration Process [Download](/individual_iFlows/_2_LocalIntegrationProcess_Weather_and_Forecast.zip).
So in the first ario I have used all the apis in a single iFlow and it looks very complex. This time we divide the iFlow into smaller chunks using local integration process. The scenario is getting current weather data and 5 day forecast directly sent to your mail!

1. Use two process calls in the main iFlow.
2. In the first local integration process fetch current weather details and store both the weather details and latitude and longitude.
3. Use second local integration process to get 5 days forecast using the latitude and longitude as query values
4. Convert both to XML and combine and send to mail.
5. Use exception subprocess to catch errors

### 3. Batch Processing Invoices [Download](/individual_iFlows/_3_Batch_processing.zip).
The task is, you will receive a batch of multiple invoices in a single combined XML file named Invoice. You have to split the invoices individually and assign severity to them. If amount is more than 10,000 severity is High. Any location other than India and America will be passed to Invalid Severity. Rest is Normal severity. Also add some data validation, if there are empty fields, replace it with 'MISSING' and set its severity to Invalid. Based on the severity send it to different mails.

Input XML structure:
```xml
<Invoices>
    <Invoice>
        <InvoiceId>INV-001</InvoiceId>
        <CustomerName>Amit Kumar</CustomerName>
        <Region>IN</Region>
        <Amount>25000</Amount>
    </Invoice>
    <Invoice>
        <InvoiceId>INV-002</InvoiceId>
        <CustomerName>John Smith</CustomerName>
        <Region>US</Region>
        <Amount>3500</Amount>
    </Invoice>
    <Invoice>
        <InvoiceId>INV-003</InvoiceId>
        <CustomerName>Lisa</CustomerName>
        <Region>EU</Region>
        <Amount>2000</Amount>
    </Invoice>
</Invoices>
```
1. Split the invoices using a splitter (general/iterative).
2. Use groovy script to assign severity function.
3. Route based on severity (High, Normal and default is Invalid).

### 4. Logging weather details in an SAP HANA table using JDBC [Download](/individual_iFlows/_4_jdbc_hana_weather_table.zip).
You will pass a city name through postman. The api with fetch the current weather details and log it to weather details table in SAP HANA. Also log the error details.

1. Fetch weather details using openweather api.
2. Convert the JSON response into XML and store relevant data into properties.
3. Use a content modifier to set the query for inserting into the table.

Content Modifier message body query:

```xml
<root>
    <Statement>
        <dbTableName action="INSERT">
            <table>WEATHER_LOGS</table>
            <access>
                <CITY>${property.cityname}</CITY>
                <TIMEZONE>${property.time}</TIMEZONE>
                <TEMPERATURE>${property.temp}</TEMPERATURE>
                <FEELSLIKE>${property.feels}</FEELSLIKE>
            </access>
        </dbTableName>
    </Statement>
</root>
```
Do same for error logs table. The tables should look something like this:

![Weather Table](/assets/weather_table.png)
*Table showing current weather logs*.

![Weather Table](/assets/error_logs.png)
*Table showing error logs*.

### 5. Processing pending requests in an SAP HANA table using JDBC [Download](/individual_iFlows/_5_jdbc_update_Pending_requests.zip).
Suppose your business has a table with products and prices in different currencies. Your business wants them in INR, so any currency thats not INR shows the status as pending, the INR ones show as processed.
The table looks somethiing like this:

![Pending Requests Table](/assets/requests_pending.png)
*Table showing pending requests*.

1. Read the table, using a content modifier and the below query.
```xml
<root>
    <Statement>
        <dbTableName action="SQL_QUERY">
            <access>SELECT PRODUCTID, PRODUCTNAME, PRICE, BASE_CURRENCY FROM GLOBAL_PRODUCTS WHERE STATUS = 'PENDING'</access>
        </dbTableName>
    </Statement>
</root>
```
2. Split all the rows of the table using a general or iterative spplitter.
3. Fetch individual values of the rows and using an API convert the currency into INR.
4. Use another content modifier to update the currency.
```xml
<root>
    <Statement>
        <dbTableName action="SQL_DML">
            <access>
                UPDATE GLOBAL_PRODUCTS SET PRICE = ${property.newAmount}, STATUS = 'PROCESSED' WHERE PRODUCTID = ${property.pid};
            </access>
        </dbTableName>
    </Statement>
</root>
```
The result table should look like this:

![Processed Table](/assets/requests_processed.png)
*Table showing the result after processing*.

### 6. Performing CRUD operations in an SAP HANA table using JDBC [Download](/individual_iFlows/_6_jdbc_CRUD.zip).
We have already done, INSERT, READ and UPDATE operations. How about doing all of them in a single iFLow!

Use this as your input XML:
```xml
<root>
    <ACTION>INSERT</ACTION> <!-- READ, INSERT, UPDATE, DELETE -->
    <TABLE>NEW_PRODUCTS</TABLE>
    <!-- READ, UPDATE, DELETE -->
    <IDname>PRODUCTID</IDname> 
    <IDvalue>104</IDvalue>
    <!-- READ -->
    <READ>PRICE, PRODUCTNAME</READ>
    <!-- INSERT -->
    <COL>PRODUCTID, PRODUCTNAME, CATEGORY, PRICE</COL>
    <VAL>104, 'Desk', 'Furniture', 299.99</VAL>
    <!-- UPDATE -->
    <SET>PRICE = 121.50</SET>
</root>
```

The table where we will be performing CRUD operations should look like this:

![CRUD table](/assets/crud_table.png)
*Dummy table showing product details*.

1. Use a content modifier and fetch the ACTION tag.
2. Route the message using the action tag to.
3. Add a content modifier after each route to perform the specified task (INSERT, READ, UPDATE and DELETE).

Queries for specified actions:
```xml
<root>
    <Statement>
        <dbTableName action="SQL_DML">
            <access>
                INSERT INTO ${property.table} (${property.col}) VALUES (${property.val})
            </access>
        </dbTableName>
    </Statement>
</root>
```
*Perform Insert Operation*.

```xml
<root>
    <Statement>
        <dbTableName action="SQL_QUERY">
            <access>
                SELECT ${property.display} FROM ${property.table} WHERE ${property.keyn} = ${property.keyv}
            </access>
        </dbTableName>
    </Statement>
</root>
```
*Perform Read Operation*.

```xml
<root>
    <Statement>
        <dbTableName action="SQL_DML">
            <access>
                UPDATE ${property.table} SET ${property.set} WHERE ${property.keyn} = ${property.keyv}
            </access>
        </dbTableName>
    </Statement>
</root>
```
*Perform Update Operation*.

```xml
<root>
    <Statement>
        <dbTableName action="SQL_DML">
            <access>
                DELETE FROM ${property.table} WHERE ${property.keyn} = ${property.keyv}
            </access>
        </dbTableName>
    </Statement>
</root>
```
*Perform Delete Operation*.

### 7. Getting Country info using SOAP adapter [Download](/individual_iFlows/_7_SOAP_Country_Info.zip).
Pass Country name in a SOAP envelope and fetch its details!

SOAP Envelope structure:
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://www.oorsprong.org/websamples.countryinfo">
   <soapenv:Header/>
   <soapenv:Body>
      <web:CountryISOCode>
        <web:sCountryName>India</web:sCountryName>
      </web:CountryISOCode>
   </soapenv:Body>
</soapenv:Envelope>
```

1. Use a SOAP sender adapter.
2. Make sure to upload the SOAP WSDL file. [Get it From here](/input_files/soap.wsdl).
3. Fetch country code.
4. Use country code to fetch entire country details.
5. Store the values of the response into individual properties and create an XML file and store it to an SFTP server.

The XML file should look something like this:
```xml
<root>
    <code>${property.Country_ISOCode}</code>
    <name>${property.Country_Name}</name>
    <capital>${property.Country_CapitalCity}</capital>
    <phone>${property.Country_PhoneCode}</phone>
    <continent>${property.Country_ContinentCode}</continent>
    <curr>${property.Country_CurrencyISOCode}</curr>
    <langc>${property.Language_ISOCode}</langc>
    <langn>${property.Language_Name}</langn>
</root>
```

### 8. Data Extraction and Storing in an SFTP server [Download](/individual_iFlows/_8_SFTP_Data_Extraction_and_Storing.zip).
We created an XML file in the previous ario, now it's time to extract its details and store it in individual folders within SFTP.

1. Start by fetching the XML file from the SFTP server.
2. Fetch all the data and store it in individual properties.
3. Use multicast and sent the data to specific routes.
4. Add content modifiers in those routes and put the values in the message body.
5. Now use SFTP receiver adapters to create directories if it doesn't exist and move the extracted files with specific values there.

# Closing thoughts
We are finally done with all our iFlows. That was quite a journey and I hope you had fun experimenting with these scenarios! When I was still learning before my career started in CPI, I tried implementing all the functions and adapters I learned, no specific scenario, just put everything together in a very messy iFLow. Now, that I kind of have an idea of how real world scenarios work I started re-implementing all the knowledge in actual business like scenarios. This should serve as a good exercise to further improve your CPI skills on actual business scenarios instead of just grouping many functions and adapters together. I wanted to share this because finding realistic practice material was a huge hurdle for me early on. I hope these examples save you some of that frustration!