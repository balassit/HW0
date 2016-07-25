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
  
