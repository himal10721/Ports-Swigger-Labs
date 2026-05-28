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
In-band SSRF: 
Easier to expploit because it shows the response
If the application allows for user-supplied arbitrary URLs, we can try the following attacks:
- Determine if a port number can be specified
- If successful, we can attempt to port-scan the internal network using Burp Intruder
- Attempt to connect other services on the loopback address
- DNS Rebinding: registering a domain name that resolves to internal IP address (DNS Rebinding). It checks if the domain has an internal IP address. 
- HTTP Redirection: Use a URL that points to a server that we control. The server has a public IP address and once the URL is requested or visited Iit redirects the vulnerable server to make a req to internal service.
- Exploit inconsistencies in URL parsing

Blind or Out-of-Band SSRF
-Attempt to trigger an HTTP request to an external system that we control and monitor the system for network interactions from the vulnerable server

### Lab: Basic SSRF against the local server
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.

1. First we try to access the admin page, but its not accessible
![alt text](../images/image-186.png)

2. We go to a product and check stock.
![alt text](../images/image-187.png)

3. In burp, we need to investigate the process. Below is the investigation process for Burp.
![alt text](../images/image-188.png)
We can see that there is an API call being made.

4. Lets try to change the URL in the sotckApi paramter to localhost

![alt text](../images/image-189.png)

5. After changing it we can access the admin page. Lets delete carlos user and complete the lab.

But before that, in order to delete carlos, we need to find out the URL to delete it. We check in the HTML tags.

![alt text](../images/image-190.png)

6. After submitting it, we solve the lab.
![alt text](../images/image-200.png)
