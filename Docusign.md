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

1.)	Get Login Info

    a.	Connection has url https://demo.docusign.net/restapi/v2 (the base URL), there is no account ID, yet. 
    b.	Operation appends ‘/login_information?api_password=true&include_account_id_guid=true&login_settings=all’ which will retrieve the account ID that will be used for the remaining steps. Other information is available, but not currently used.
    c.	The account_id is saved in a dynamic process property, so that it is appended in each remaining step.
    
2.)	Get Envelopes 

    a.	https://demo.docusign.net/restapi/v2/{account_id}/envelopes?from_date={year-month-day}&status={status}
    b.	Retrieves a list of the envelopes based on the query provided. Currently, the query is a static string set in the parameters list. This is where you would choose from the list of statuses mentioned above, for envelopes.
    c.	Each envelope has a list of URIs that are saved to be used in the remaining steps. The Envelope Uri is mapped to a flat file, so that it can be referenced in the proceeding steps. 
    d.	A split on the envelope ID is used to run each envelope through the proceeding operations.

3.) A branch is used in case the process is intended to execute all sub processes. However, it may be of best practice to separate the processes.

##Docusign Get Envelope

1.) Get Envelope
    
    a. Retrieves Uris for a specific envelope. This list includes the documents/recipients/envelope/templates/certificate uri and the envelope ID.
    b. Saves each uri as a dynamic process property so that the following operations can use them. 
    
2.) Get Document List
    
    a. Retrieves a list of all documents in this envelope.
    b. Saves the evelope ID, name of document, and uri from the "Docusign Document Response JSON Profile" to the "Document Names Flat File Profile" via Document Names Map
    c. The purpose of this steps is to route the document to the correct subprocess, so the fields of the document match the correct JSON profile.
    d. The route is based on the name of the document, and checked against hard coded strings for the completed document JSON profiles and database tables that have been completed thus far.
    
4.)	Get Envelope Recipients

    a.	?include_tabs=true is appended as a static value to obtain all tabs of the document. 
    b.	Important fields: Name (type), value, document id, recipient id, tab id, signedDateTime, deliveredDateTime, status, recipient count, tabLabel
5.)	Get Documents Combined

    a.	Returns a pdf version of all of the documents in this envelope.
    b.	Static parameter ‘?show_changes=true’ can be used to highlight the fields that were changed. 
6.)	Get Documents

    a.	Using the split shape, get all envelopes one at a time.
    b.	Get the list of documents in each envelope, then obtain the uri for each one, and save it flat file with the name of the document, and the document id.
7.)	Delete Recipient

    a.	Uses Envelope Combined Documents profile, from #5 to append the recipient id to the url. 
    b.	If the recipient does not exist, it will fail. This shouldn’t be a problem, since the value is obtained from looking up the recipient id in a prior step, with no modifications in between. 
    c.	This will only pull the first recipient id from the file. There needs to be a modification to get all recipient ids, and run delete recipient on each one, or a subset.





##Docusign Get Documents
  

