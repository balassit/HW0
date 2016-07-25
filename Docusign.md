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

###Get Login Info:

1.)	Get Login Info

    a.	Connection has url (https://demo.docusign.net/restapi/v2), there is no account ID, yet. 
    b.	Operation appends ‘/login_information?api_password=true&include_account_id_guid=true&login_settings=all’ which will retrieve the account ID that will be used for the remaining steps. Other information is available, but not currently used.
    c.	The account_id is saved in a dynamic process property, so that it is appended in each remaining step.
2.)	Get Envelopes 

    a.	https://demo.docusign.net/restapi/v2/{account_id}/envelopes?from_date={year-month-day}&status={status}
    b.	Retrieves a list of the envelopes based on the query provided. Currently, the query is a static string set in the parameters list. 
    c.	Each envelop has a list of Uris that are saved to be used in the remaining steps. The Envelope Uri is mapped to a flat file, so that it can be referenced.
    d.	A split on the envelope ID is used to run each envelope through the following operations.
3.)	Get Envelope

    a.	Retrieves Uris for a specific envelope. 
    b.	Saves each uri as a dynamic process property so that the following operations can use them. 
    c.	This operation is the setup for all others.
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
  

