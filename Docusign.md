#Docusign Process

Location: Boomi Directory Docusign, under Boventus.

Main Process: Docusign

Sub-processes:

##Active
    
    1. Docusign Get Documents 
    2. Docusign Get Envelope Fields Exogen
    3. Docusign Get Envelope Fields SAP
    4. Docusign Get Envelope Recipients
    
##Inactive
    
    5. Docusign Password Grant and API Call
    6. Docusign Get Documents Combined
    7. Docusign Add Envelope Recipients
    8. Docusign Delete Recipients
    
#General Connection Information

Our goal is to connect to DocuSign RESTful calls in Boomi, using HTTP Connections with an authentication token.
The Docusign Rest API Guide Version 2 details the requirements.

http://iodocs.docusign.com/ can be used to send REST calls, and ensure that they work correctly outside of Boomi. The nexwer version of the apiexplorer is more helpful, and helps to generate the correct request and response information, along with access to information about each call.

An integrator key is unique to each integration, and is used for all API calls. The current integrator key is used in the demo environment, but can be authorized for producion use. The integrator key can be found by going to:

    https://appdemo.docusign.com/home --> Admin --> APIs and Keys
A list of integrator keys is shown. A secret key can be generated for each integrator key, but is only visible once. It is important to save this value on creation. A secret can be deleted and recreated, if the prior was lost. 

Docusign can use JSON or XML for the request and response formats. The default is json, but can be changed by appending ".json" or".xml" to the end of a resource call. 

Ex. https://preview.docusign.net/restapi/v2/accounts/{accountId}/envelopes.json

Envelope status codes (page 15 of the API Guide)

    1. Created
    2. Deleted
    3. Sent
    4. Delivered
    5. Signed
    6. Completed
    7. Declined
    8. Voided
    9. Timed out
    10. Authritative Copy
    11. Transfer Completed
    12. Template
    13. Correct

There are also Recipient Status Codes listed below the Envelope Status Codes.

#Walkthrough of processes

##Docusign

The authroization header must be included in all calls. It is saved as a process property, and does not have tochange between executions. Each operation has the authorization header as a parameter. The autorization header can be revoked and reset. The production environment only allows 10 to be generated and kept active, so it is important to use few and save their values.

1.	Get Login Info

    a.	Connection has url https://demo.docusign.net/restapi/v2 (the base URL), there is no account ID, yet. 
    b.	Operation appends ‘/login_information?api_password=true&include_account_id_guid=true&login_settings=all’ which will retrieve the account ID that will be used for the remaining steps. Other information is available, but not currently used.
    c.	The account_id is saved in a dynamic process property, so that it is appended in each remaining step.
    
2.	Get Envelopes 

    a.	https://demo.docusign.net/restapi/v2/{account_id}/envelopes?from_date={year-month-day}&status={status}
    b.	Retrieves a list of the envelopes based on the query provided. Currently, the query is a static string set in the parameters list. This is where you would choose from the list of statuses mentioned above, for envelopes.
    c.	Each envelope has a list of URIs that are saved to be used in the remaining steps. The Envelope Uri is mapped to a flat file, so that it can be referenced in the proceeding steps. 
    d.	A split on the envelope ID is used to run each envelope through the proceeding operations.

3. A branch is used in case the process is intended to execute all sub processes. However, it may be of best practice to separate the processes.

##Docusign Get Documents

1. Get Envelope
    
    a. Retrieves Uris for a specific envelope. This list includes the documents/recipients/envelope/templates/certificate uri and the envelope ID.
    b. Saves each uri as a dynamic process property so that the following operations can use them. 
    
2. Get Document List
    
    a. Retrieves a list of all documents in this envelope.
    b. Saves the evelope ID, name of document, and uri from the "Docusign Document Response JSON Profile" to the "Document Names Flat File Profile" via Document Names Map
    c. The purpose of this steps is to route the document to the correct subprocess, so the fields of the document match the correct JSON profile.
    d. The route is based on the name of the document, and checked against hard coded strings for the completed document JSON profiles and database tables that have been completed thus far.

3. Get Document

    a. Returns a pdf version of the documents in the envelope. This sub process stops here, and has not sent the pdf file to a location. The pdf may be sent to sharepoint, or another destination, at another time. 
    b. Other part of the branch redirects to a subprocess to obtain the fields of the document.
    
##Docusign Get Envelope Fields Exogen/SAP

1. Get Envelope Recipients

    a. Obtains a JSON file of the fields of the specified document
    b. Goes to a branch to map the content of the file. The fields that are saved include: tabLabel, documentId, EnvelopeId, and value. The envelope Id is obtained through a get dynamic process property, which goes back to when it was saved in the previous process under Get Document List. 

2. Mappings and Branches

    a. The first branch, "Recipient ID Exogen/SAP FOrm" is a cache to save all recipient IDs from the JSON file. The recipient IDs are used in the mapping for signature and initials, so that the value is the name/email combination for that recipient ID. The "value" field for signature and initial tabs is either 1 or 0, for true or false, which does not provide much valuable information.
    b. The date signed tab can ensure that the document was signed by the recipient, and the name/email combination determines who it was. Name alone is not sufficient, because more than one person could have that name.
    c. Each branch sends an insert statement through the BOBJSQL connection to the [BV_STG].[dbo].[zDocusign_Document_Temp] database table. This can be seen in the Docusign Form Fields Database Profile. 
    d. The last branch clears the envelope ID from the dynamic process proerty, so that it is not used in the following executions. There was a problem with gauranteeing that the same envelope ID was not used later, but with testing, this branch may be removed.
    e. The update stored procedure step runs a stored procedure to pivot the data and place it in the correct final table. More information on this step is provided in the database section.

##Datebase

1. Inital Load

    a. The first table populated is the [BV_STG].[dbo].[zDocusign_Document_Temp] which has columns for  tabLabel, documentId, EnvelopeId, and value. This table can be used for all docusign forms, since each form has the same valueable information. The tabLabel is the description of a field on the form. The document ID and envelope ID are used to identify the document, and the value is the value of the field from the form.
    b. The [BV_STG].[dbo].[zDocusign_Document_Temp] is assumed to be created already, and is not created on execution of a Boomi process. If the table is dropped, then the database inserts would fail. 
    c. After the data is inserted in this table, the stored procedure for the specific document type is executed.
    
2. Stored Procedure
    a. sp_Docusign_Exogen_Patient_Assistance_Form
    b. sp_docusign_Global_SAP_Security_Acess_Form
Each stored procedure starts by dynamically pivoting the [BV_STG].[dbo].[zDocusign_Document_Temp] table on the tabLabel and value. This steps make the rows for the tabLabel into column headers, and the value is placed under each column. By doing this, a tabel is created with one row, and however many columns are needed, based on the number of tabLabels. This section can be copied for all docusign database loads, and eases the process of generating the table data. The tabel created is [BV_STG].[dbo].[zdocusign]

3. [BV_STG].[dbo].[zdocusign]
    a. This table is temporarily used to merge the incoming data with the final table for the document type. The data cannot be directly inserted into the final table, because it may already exist, and duplciates should not be made. 
    b. The merge or insert step for each document type will be different, and manually created. There is no way to dynamically merge or insert. The format will remain the same, but the values from tabLabel will be different, which means that the coolumn names will differ. I merge when the envelope and document id are identical, which should ensure that that the same document is being updated, rather than a new one created. The target table is [zDocusign_SAP_Form] or whatever the document type is, and the created table for it. The source data is from the [BV_STG].[dbo].[zdocusign] table. 
    c. After mergining or inserting the row into the correct table, the temporary tabel [BV_STG].[dbo].[zdocusign] is dropped. Note that it most be dropped the column names may differ on the next execution. The [BV_STG].[dbo].[zDocusign_Document_Temp] is deleted, which only deletes the data from the table, but not the format of the columns.
    
    
    
    
    
4.	Get Envelope Fields

    a.	?include_tabs=true is appended as a static value to obtain all tabs of the document. 
    b.	Important fields: Name (type), value, document id, recipient id, tab id, signedDateTime, deliveredDateTime, status, recipient count, tabLabel
5.	Get Documents Combined

    a.	Returns a pdf version of all of the documents in this envelope.
    b.	Static parameter ‘?show_changes=true’ can be used to highlight the fields that were changed. 
6.	Get Documents

    a.	Using the split shape, get all envelopes one at a time.
    b.	Get the list of documents in each envelope, then obtain the uri for each one, and save it flat file with the name of the document, and the document id.
7.	Delete Recipient

    a.	Uses Envelope Combined Documents profile, from #5 to append the recipient id to the url. 
    b.	If the recipient does not exist, it will fail. This shouldn’t be a problem, since the value is obtained from looking up the recipient id in a prior step, with no modifications in between. 
    c.	This will only pull the first recipient id from the file. There needs to be a modification to get all recipient ids, and run delete recipient on each one, or a subset.





##Docusign Get Documents
  

