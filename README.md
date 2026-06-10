# Web API Training Assignments

This README lists all the assignments for the Web API Framework training. For this training it is mandatory that you run a WEB project. This will not work for windows or flextron applications.

## Assignment 1: Build Your First Endpoints

Goal: set up first Web API endpoints.

Steps:
- Set up your cWebApi (File -> New -> Server / Service objects).
- Add two endpoints.
- Use two separate `cRestDataset` objects.
- Put each endpoint in its own `cWebApiRouter`.
- Make one endpoint use a `cRestEntity` for a parent relationship.
- Make one endpoint use a `cRestChildCollection` for a child relationship.
- Make one endpoint contain a field whose value is returned by `OnSetCalculatedValue`.

## Assignment 2: Implement Swagger UI Documentation

Goal: add generated API documentation and test it.

Steps:
- Implement Swagger UI in a new view using (File -> New -> Views -> Data entry).
- Add to your view `Use WebApi\cSwaggerUI.pkg`.
- Open the generated view in the browser.
- Test the `Try it out` feature.
- Experiment with parameters.
- Change `cRestDataset` properties such as `pbReadOnly` and `pbAllowEdit`.
- Change `cRestField` properties such as `pbWriteOnly` and `pbFilterable`.

## Assignment 3: Build a Custom Endpoint

Goal: create and document a custom Web API endpoint.

Steps:
- Implement a `cWebApiCustomEndpoint`.
- Start with returning `{"message": "Hello world!"}` in `OnHttpGet`.
- Implement `OnDefineSchema` so Swagger UI shows the proper response.
- Add a `Name` query parameter in `OnDefineSchema`.
- Change the response so it includes the provided name, for example `{"message": "Hello world, Niels!"}`.

Bonus:
- Try to get file upload working in `OnHttpPost`.
- Store uploaded file data in `data/uploads`.

Information about OnDefineSchema:

The `tEndpointDefinition` holds a `tVerbDefinition` array. You can define as many of these as you want inside of the `tEndpointDefinition`. For the steps described above we would only need 1. Maybe 2 if you also intend to do the bonus step.

The `tVerbDefinition` actually describes what your verb is supposed to do by specifying the following
- The sVerb should be GET, POST, PUT, PATCH or DELETE
- sDescription specifies the description that will be shown inside of the documentation
- tResponseDefinition will hold what the response will be
- asRequestMediaTypes is only needed for POST, PUT and PATCH. It specifies what media types we acccept, such as application/json, application/xml.
- tFieldDefinition will hold the fields we expect the caller to pass in the request body. This is only needed for POST, PUT and PATCH.
- tParameterDefinition defines the information about our parameters, these can either be parameters passed in the header or in the query. For the first step you will need a query parameter.

The `tFieldDefinition` holds the information of a field. Both request and responses make use of this definition.
- the sFieldName is the actual name of the field shown in the documentation
- the sExampleValue is what will be shown to the user when viewing a POST, PUT and PATCH request in the documentation. They are like dummy values.
- eFieldType specifies the actual type of the field. You can check the enumlist in the struct to see what is available.
- bRequired marks this field as required inside of the specification
- the nestedField members are important for when you want to specify something like a json response. You would make a tFieldDefinition that is of eFieldType WEBAPI_OBJECT_FIELD, and then the nested fields can be the JSON members that should be inside of this JSON object. For example, the nested field in the steps listed above would be a tFieldDefinition with eFieldType set to WEBAPI_STRING_FIELD and sFieldName set to "message"!

The `tResponseDefinition` holds the response information. One verb can hold multiple tResponseDefinitions, this is because a endpoint can return multiple results. For example it can return a status code 200 OK when everything went right. But it could also return a error code like 404, 403 FORBIDDEN.
- iStatusCode determines the status code that will be shown inside of the swagger documentation. You can find information about the status codes here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status#successful_responses
- sStatusCodeDescription is a description for the status code, usually you match this with what is shown in the specification above. For example, if your iStatusCode is 200, your sStatusCodeDescription would often be something like "OK".
- asResponseMediaTypes holds a list of media types this request could return, for example application/json, application/xml. For the steps above we would only need application/json.
- responseFields will determine what the actual response will look like in the documentation. This works in a similar way as to how it works for the tFieldDefinition struct shown above.

The `tParamaterDefinition` holds the information about all parameters that can be send with the request. This can either be query parameters (so inside of the URL), or header parameters passed as a header during the call.
- sName describes the name of the parameter.
- sDescription holds a short description of what the parameter is about.
- eParameterIn determines where the parameter will be located, this can either be WEBAPI_QUERY_PARAMETER to have it be in the query (which is what we will need for the steps provided in the assignment) or a WEBAPI_HEADER_PARAMETER.
- eParameterType determines the type of the parameter such as a string, integer, number, etc. You can find the available values in the enum list.
- bRequired determines if the parameter should be required.




## Assignment 4: Implement Logging

Goal: log incoming API requests.

Steps:
- Implement request logging with a `cWebApiModifier`.
- Create a new `Log` table.
- Log incoming request time.
- Log the HTTP verb (Hint, webapicallcontext!).
- Log the time taken.
- Log the status code.
- Add any other useful request information you want.

## Assignment 5: Implement Basic Auth

Goal: add authentication and simple authorization.

Steps:
- Implement `BasicAuth`.
- Use the Basic Auth specification from Swagger documentation .
- Start implementing the OnAuth event.
- Read the `Authorization` header.
- Remember that the authorization header is seperated by a space "Authorization bad13123da=". We only need the part after the space. (Tip, use StrSplitToArray OR Right in combination with Pos here!)
- Base64Decode the string behind the space.
- Split the string at the ":". The part before the ":" is the username, the part behind it will be the password.
- Check whether the user exists and check if the password matches the password in the database (VerifyPasswordHash of ghoWebSessionManager), do not actually login and do not create a session.
- Implement role-based access.
- Only allow admin users to delete resources.
- Check Swagger UI after implementation.
- Try different positions for the `cWebApiAuthModifier`: global (cWebApi), router (cWebApiRouter), and endpoint (cRestDataset) level.

Sample Base64 decode code:

```df
//Decode the info in the Authorization header
Move (Base64Decode(AddressOf(asAuthorizationParts[1]), &iLength)) to pBaseDecodedString
Move (Repeat(Character(0), iLength)) to sDecodedString
Move (MemCopy(AddressOf(sDecodedString), pBaseDecodedString, iLength)) to iVoid
Move (Free(pBaseDecodedString)) to iVoid

//First element will be the username and the second element will be the password
Get StrSplitToArray sDecodedString ":" to asDecodedUsernamePassword
```

To grab the authorization header
```df
Get HttpRequestHeader "Authorization" to sBase64EncodedAuth
```

Remember to set webapicallcontext.bErr to true when the current request is NOT authenticated. This tells the framework to not process the event any further.
``` df
If (not(bOk)) Begin
    Move True to webapicallcontext.bErr
    Move C_WEBAPI_FORBIDDEN to webapicallcontext.iStatusCode
    Move "Forbidden" to webapicallcontext.sShortStatusMessage
    Move "You do not have sufficient permissions to access this resource" to webapicallcontext.sErrorMessage
End
```

The following code shows how to perform the user check without logging in. Try to only copy this code if you get stuck
```df
//First element will be the username and the second element will be the password
Get StrSplitToArray sDecodedString ":" to asDecodedUsernamePassword

// Ensure we start with an empty state
Send Clear of oWebAppUserDD
Move asDecodedUsernamePassword[0] to WebAppUser.LoginName
Send Find of oWebAppUserDD EQ 1

If (Found) Begin
    Get Field_Current_Value of oWebAppUserDD Field WebAppUser.Password to sUserPassword
    Get VerifyPasswordHash of ghoWebSessionManager (Trim(sUserPassword)) (Trim(asDecodedUsernamePassword[1])) (&bUpgrade) to bOk        
End
Else Begin
    Move False to bOk
End
```

## Assignment 6: Implement the MCP Module

Goal: expose datasets through MCP tooling.

Steps:
- Download the MCP module from the package manager.
- Start by changing one `cRestDataset` into a `cMcpRestDataset`.
- If available, try integrating it with tools such as Codex or Claude Code through MCP server settings.
- When you have your MCP server connected, try and think about endpoints that you would like to expose to the customer that could be useful.
- If you do not have AI tooling installed, you can try installing codex desktop from the microsoft store and checking if the free-tier allows MCP servers to be added.

To add your MCP server to codex do the following:
- Go to settings.
- Under integrations, select "MCP servers".
- Press "Add server".
- Select the "Streamable HTTP" option and not "STDIO".
- In the url, insert the url to your mcp endpoint. For example: http://localhost/WebAPITraining/Api/mcp
- Press save
- To test, open a new chat and type /mcp in the chat bar. This should show your mcp server.
- Ask the AI a question like "Give me a list of all RESOURCE YOU EXPOSED" and see if it returns it.

Claude steps:
- Claude seems to not have a automatic way to add localhost MCP servers. Refer to the step listed below.

We've seen situations where AI agents do not pick up on MCP servers hosted on localhost, if this is the case you can explicitly mention the mcp server in the chat. Say something like "My mcp server is hosted at http://localhost/WebAPITraining/Api/mcp, please show me all available endpoints".
