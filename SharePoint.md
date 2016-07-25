Current connections: zBlake-Test--> SharePoint

SharePoint through HTTP connection with basic authentication:
    [link][1] provides steps and screenshots of this process
	[link][5] provides url endpoints for REST calls 

    1)	Credentials: currently given full control of crmuser account
        i.	Username
        ii.	Password
    2)	Get Authentication
        i. Obtains the Form Digest Value, which is a long hex string. This value persists for 30 minutes. 
    3) Get List Data
    4) Modify Item in List
	5) Boomi Community
	6) Previous Information

2) Get Authentication using Postman:

    POST
    URL: https://bioventus.sharepoint.com/_api/contextinfo
	
    Basic Authentication using username and password
    Headers: 
        Accept: application/json;odata=verbose
        Content-Type: application/x-www-form-urlencoded; charset=UTF-8
        Origin: https://bioventus.sharepoint.com
        Authorization: Basic {access token} (this value was being set in Postman on SEND.
    Response: 
    see attachment --> Authentication.json
    {
      "d": {
        "GetContextWebInformation": {
          "__metadata": {
            "type": "SP.ContextWebInformation"
          },
          "FormDigestTimeoutSeconds": 1800,
          "FormDigestValue": "0x67422A15C0FDDAA516D4345DE451A6D54C69AB497AA7889BC1CBDF19538AFF23351BFDF6838C2147DC9E2BFEB9CB67289C0FF0EA0B0400965C218311066D240C,22 Jul 2016 17:35:00 -0000",
          "LibraryVersion": "16.0.5507.1212",
          "SiteFullUrl": "https://bioventus.sharepoint.com",
          "SupportedSchemaVersions": {
            "__metadata": {
              "type": "Collection(Edm.String)"
            },
            "results": [
              "14.0.0.0",
              "15.0.0.0"
            ]
          },
          "WebFullUrl": "https://bioventus.sharepoint.com"
        }
      }
    }

3) Get List Data: URL can append "('1')" to get the information of only the first element)
 
    GET
    URL: https://bioventus.sharepoint.com/GIS/Project Management/_api/web/lists/GetByTitle('boomi_cars')/Items
    Basic Authentication using username and password
    Headers:
        X-RequestDigest:0x67422A15C0FDDAA516D4345DE451A6D54C69AB497AA7889BC1CBDF19538AFF23351BFDF6838C2147DC9E2BFEB9CB67289C0FF0EA0B0400965C218311066D240C
        Accept:application/json;odata=verbose
        Content-Type:application/json;odata=verbose
        Authorization: Basic {access token} (this value was being set in Postman on SEND.
		(In some frameworks you’ll have to specify that the content-length of the POST request is 0)

    Response:
    see attachment --> Get List Item.json

[link][2] to information about creating, modifying, and deleting from a list

4) Modify Item in List

    POST
    URL: https://bioventus.sharepoint.com/GIS/Project Management/_api/web/lists/GetByTitle('boomi_cars')/Items
    Basic Authentication using username and password
    Headers:
        X-RequestDigest:0x67422A15C0FDDAA516D4345DE451A6D54C69AB497AA7889BC1CBDF19538AFF23351BFDF6838C2147DC9E2BFEB9CB67289C0FF0EA0B0400965C218311066D240C
        Accept:application/json;odata=verbose
        Content-Type:application/json;odata=verbose
        X-HTTP-Method: MERGE
        IF-MATCH: *
        Authorization: Basic {access token} (this value was being set in Postman on SEND.

    Response: 403 Forbidden Error (The request was legal, but the server is refusing to respond to it. Unlike a 401 Unauthorized response, authenticating will make no difference.
[link][3] to information about access issues. May not be relevant, since we have full control. This is not available in the bioventus account site settings

More [help][4] on SharePoint through Postman with examples

Attempts to run on Boomi have resulted in (400) - Error message received from Http Server, Code 400: Bad Request; 
on the Get List connection. Authentication successfully connects, but no data is produced. I have formatted the expected JSON file as a response to the connection as well.

	5) Boomi Community: 
	
	    i) [HTTP Client with oauth 2.0 to Office 365][6]
		ii) [Connecting to SharePoint][7]
		iii) [Any Pointers on Sharepoint integration using dell boomi ??][8]
	
    6) Previous Information:
	
        i) [Configure basic authentication][9] for claims-based web application in sharepoint
        Basic authentication enables a web browser to provide credentials when the browser makes a request during an HTTP transaction. Because user credentials are not encrypted for network transmission but are sent over the network in plaintext, we do not recommend that you use basic authentication over an unsecured HTTP connection. To use basic authentication, you should enable Secure Sockets Layer (SSL) encryption for the web site; otherwise, the credentials can be intercepted by a malicious user.
        ii)	[Turn IIS on][10]
	        a) Includes link to how to get access token
        iii)[Client Secret replacement][11]
            a) Download MS Online Services Sign-in Assistant. [Install Azure Active Directory Module][12]
        iv) [Access Token][14]


  [1]: http://tech.bool.se/basic-rest-request-sharepoint-using-postman/
  [2]: http://blog.cloudshare.com/blog/2012/12/16/access-and-manipulate-data-in-your-cloudshare-sharepoint-2013-farm-from-anywhere-with-csom-rest-and-odata
  [3]: http://www.ceus-now.com/sharepoint-2013-and-anonymous-access-problem/
  [4]: https://www.helloitsliam.com/2016/02/04/postman-and-office-365/
  [5]: https://msdn.microsoft.com/en-us/magazine/dn198245.aspx
  [6]: https://community.boomi.com/message/8201 
  [7]: https://community.boomi.com/message/6799
  [8]: https://community.boomi.com/message/6168
  [9]: https://technet.microsoft.com/en-us/library/gg576953.aspx
  [10]: http://windows.microsoft.com/en-us/windows-8/internet-information-services-iis-8-5
  [11]: https://msdn.microsoft.com/en-us/library/office/dn726681.aspx
  [12]: https://technet.microsoft.com/library/jj151815.aspx#bkmk_installmodule
  [13]: https://support.microsoft.com/en-us/kb/2669552
  [14]: http://stackoverflow.com/questions/11804624/how-can-i-get-an-oauth-access-token-in-sharepoint-2013
