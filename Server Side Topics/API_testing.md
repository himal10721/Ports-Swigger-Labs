# API testing

APIs enable software systems and applications to communicate and share data. API testing is important as vulnerabilties in APIs may undermine core aspects of a websites confidentiality, integrity and availability. 

All dynamic websites are composed of APIs, so classic web vulnerabilties like SQL injection could be classed as API testing. In this topic, we'll learnhow to test APIs that aren't fully used by the website front-end, with focus on RESTful and JSON APIs. 

## API recon

To begin, we should find out about API endpoints. These are locations where an API receives requests about a specific resource on its server.

Example
GET /api/books HTTP/1.1
Host: example.com

The API endpoint for this request is /api/books. This would result in an interaction with the API to retrieve a list of books from a library. Another API endpoint might be /api/books/mystery, which would retrieve a list of mystery books.

Once these kind of endpoints have been identified, we need to determine how we can interact with them. This enables us to construct valid HTTP requests to test the API.
We need to find out information about the following:

i. The input data the API processes, including both compulsory and optional parameters.
ii. The types of requests the API accepts, including supported HTTP methods and media formats.
ii. Rate limits and authentication mechanisms

## API documentation

APIs are usually documented so that developers know how to use them. Documentation can be in both human-readable and machine-readable forms. Human-readable documentation is designed for developers to understand how to use the API, It may include detailed explanations, examples, and usage scenarios. Machine-readable documentaiotn is desgined to be processed by software for automating tasks like API integration adn validation. It's written in structured formats like JSON and XML.

API documentation is often publicly avaiable , particularly if the API is intended for use by external developers. If this is the case, we should always start our recon by reviewing the documentation.

### Discovering API documentation

Even if the API documentation isnt openly available, we might still be able to access it by browsing apps that use the API.

In order to do this, we can use the Burp scanner to crawl the API. We can also browse apps manually using Burps browser. Look for endpoints that might refer to API documentation, for an example:
/api
/swagger/index.html
/openapi.json

If we identify an endpoint for a resource, we need to make sure to investigate the base path. For example, if we identify the resource endpoint /api/swagger/v1/users/123, we should investigate the following paths:

/api/swagger/v1
/api/swagger
/api

### Using machine-readable documentation

We can use a range of automated tools to analyze any machine-readable API documentation that we can find.

We can also use Burp Scanner to crawl and audit OpenAPI documentation, or any other documentation in JSON or YAML format. Additionally we can also parse the OpenAPI documentation using the OpenAPI Parser BApp. 

We can also use specialized tool to test the documented endpoints such as Postman or SoapUI

## Identifying API endpoints

We can gather a lot of information by browsing apps that use the API. This is a must do even if we have access to the API documentation, as sometimes documentation may be inaccurate or out of date.

We can use Burp Scanner to crawl the application, then manually investigate interesting attack surface using Burp's browser.

While browsing the application, we need to look for patterns that suggest API endpoints in the URL structure, such as /api/. Also we need to look out for JavaScript files. These can contain references to API endpoints that we haven't triggered via the web browser. Burp Scanner automatically extracts some endpoints during crawls, but for a more heavyweight extraction, we use the JS Link Finder BApp. We can also manually review JS files in Burp.

### Intereacting with API endpoints

After identifying APi endpoints, we interact with them using Burp Repeater and Burp Intruder. This helps observe the API's behavior and discover additional attack surface. 

As we interaction, review error messages and responses closesly. They can include information that we can use to construct a valid HTTP request.

#### Identifying supported HTTP methods

GET, PATCH, OPTIONS

An API endpoint may support different HTTP methods. So, investigate all the necessary endpoints when investigation. 
For example, the endpoint /api/tasks may support the following methods:

GET /api/tasks - Retrieves a list of tasks.
POST /api/tasks - Creates a new task.
DELETE /api/tasks/1 - Deletes a task.
You can use the built-in HTTP verbs list in Burp Intruder to automatically cycle through a range of methods.

#### Identifying supported content types

API endpoints often expect data in a specific format. They can beahve differently if different content-types are provided in the request. 
Changing the content type may enable us to:
Trigger errors that disclose useful information.
Bypass flawed defenses.
Take advantage of differences in processing logic. For example, an API may be secure when handling JSON data but susceptible to injection attacks when dealing with XML.

In order to change the content type, we modify the Content-Type header and reformat the request body accordingly. 

#### Using Intruder to find hidden endpoints

Once we have identified some initial API endpoints, you can use Intruder to uncover hidden endpoint. For an example, 

If we have identified an API endpoint for updating user information:

PUT /api/user/update

To identify other hidden endpoint, we could use the Burp Intruder to find resources with the same structure. 
Like add a /update position to the path with a list of other common functions such as delete and add.

When looking for hidden endpoints, look for common API naming conventions and industry terms. 

## Finding hidden parameters

When doing API recon, undocumented parameters can be found that the API supports. They can be used to change the application's behavior. Burp includes tools that can help you identify hidden parameter.

i. Burp Intruder enables to automatically discover hidden parameters.
ii. The "Param miner" BApp enables us to automatically guess up to 65536 param names per request. Param miner automatically guesses names that are relevant to the app, based on the info taken from the scope.
iii. The content discovery tool enables to discover content that isn't linked from visible content that can be used to browse including parameters.

## Mass assignment vulnerabilities

Mass assignment can create hidden parameters. It occurs when software frameworks automatically bind request parameters to fields on an internal object. Mass assignment may result in the application supporting parameters that aren't intended to be processed by the developer.

### Identifying hidden parameters

Mass assignement creates parameters from object fields. Hidden paramters can often be identified by manually examining objects returned by the API. 

For example, consider a PATCH /api/users/ request, which enables users to update their username and email, and includes the following JSON:

{
    "username": "wiener",
    "email": "wiener@example.com",
}
A concurrent GET /api/users/123 request returns the following JSON:

{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
This may indicate that the hidden id and isAdmin parameters are bound to the internal user object, alongside the updated username and email parameters.

Testing mass assignment vulnerabilities

To test the enumerated isAdmin parameter, we add it to the PATCH request:

{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": false,
}

Also, sending a PATCH request with an invalid isAdmin parameter value:

{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": "foo",
}

If application behaves differently, it may suggest that the invalid value can impacts the query logic, but the valid value doesn't. This can indicate that the parameter can successfully updated by the user. 

By setting the "isAdmin" : true, we might be able to access the account as an admin.
If the isAdmin value in the request is bound to the user object without adequate validation and sanitization, the user can be incorrecgtly granted admin privileges.

## Preventing vulnerabilities in APIs

When designing APIs, the following should be kept in consideration:
i. Securing the documentation if the API shouldnt be publicly accessible. 
ii. Ensuring documentation is kept up to date so that legitimate tester have full visibility of the API's attack surface.
iii. Applying an allow list of permitted HTTP methods
iv. Validate that the content type is expected for each request or response
v. using generic error messages to avoid giving away info that can be useful to an attacker
vi. Using protective measures on all version of API

In order to prevent mass assignment, we need to allowlist the properties that can be updated by the user and blocklist sensitive properties that shouldn't be updated by the user.
