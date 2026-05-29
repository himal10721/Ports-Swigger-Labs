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


### Lab: Basic SSRF against another back-end system

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal 192.168.0.X range for an admin interface on port 8080, then use it to delete the user carlos.

1. First we visit a product, and click on check stock, We then intercept the request and send it to Burp intruder.

![alt text](../images/image-201.png)

2. After sending it to repeater, we can see that the api is bein called on 192.168.0.1....
ctrl+shift+U will decode the URL parameter

![alt text](../images/image-202.png)

3. We're now checking if there's any other services running on other IP's.
![alt text](../images/image-203.png)

BUt it is not possible for us to check each and every one of it, so lets simply automate the IP check in intruder to check for 255 IP addresses.


4. In intruder, simply add the payload parameter and set the range from 1 to 255 with step 1 because there are 255 IP addresses.

![alt text](../images/image-204.png)

We got a 404 response for 192.168.0.53, which means this server is running. Lets send it to repeater and play around with it.

5. Lets see if admin page exists here, we can check it by placing /admin to the end of the URL and send the request.

![alt text](../images/image-205.png)

6. The admin page was indeed hosted on this server.

![alt text](../images/image-206.png)

We find the delete functionality in the HTML page and copy the url paste it in the stockApi paramter and send to solve the lab!

7. Follow redirection after this page.

![alt text](../images/image-207.png)

8. Lets go back to the post request and check the admin panel and see if we deleted the user

![alt text](../images/image-208.png)

Lab solved :)



### Lab: SSRF with blacklist-based input filter

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.

The developer has deployed two weak anti-SSRF defenses that you will need to bypass.

1. Lets check stock and intercept the request and send it to repeater.
![alt text](../images/image-95.png)

2. The normal response is 200 and we can also see the request URL parameter.

![alt text](../images/image-96.png)

3. When checking if there is any localhost, we get an error message called "External stock check blocked for security reasons.

![alt text](../images/image-97.png)

4. When checking for 127.1, we can see that there is a 200 response, so that mean there must be something hosted on that server.
 ![alt text](../images/image-98.png)

5. Lets open the response in browser and see what opens up
![alt text](../images/image-99.png)

Boom, there is the admin panel.

6. But we are still being block. There must be a string checker blocking the letters "admin"

![alt text](../images/image-149.png)

7. SO lets double encode the characters and see what happens

![alt text](../images/image-156.png)

It works.

8. lets copy that URL and open in browser and delete the user

![alt text](../images/image-157.png)

Lab solved! :)

![alt text](../images/image-158.png)
