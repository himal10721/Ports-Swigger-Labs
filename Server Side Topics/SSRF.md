# What is SSRF ?

SSRF is a web security vulnerability that allows an nattacker to cause the server-side application to make requests to an unintended location. 

In a typical SSRF attack, the attacker might cause the server to make a connection to internal-only services within the organizations infrastructure. 

## How it occurs?

It occurs when an app is fetching a remote resource without first validating the user supplied URL 

## Types of SSRF

i. Regular / In band: Can tamper with the URL and the response will be shown in the application. 

ii. Blind / Out-of-Band: In this, the application won't show the response back to us in the application!

## Impact:

Depends on the functionality in the application that is being exploited.
i. Confidentiality - can be None/ Partial/ High
ii. Integrity - same as above
iii. Availability- same as above

## How to find SSRF ?

i. Black Box Testing

- Mapping the application: Identify any request parameters that contain hostnames, IP addresses or full URLs
- Fuzzing :- for each request, modify its vlaue to  specify an alternative resource and observe how the application responds. 

- For blind or out-of-band, modify the value of the parameter to the server on the internet that we have control on and can monitor the server for incoming request
  


ii. White Box Testing

- Review source code and identify all request parameters that accept the URLs
- can be done by combining both black-box and white-box testing perspective

## Exploiting SSRF vulnerabilities
