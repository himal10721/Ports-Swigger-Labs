# File upload vulnerabilites

Those vulnerabilities that allows user to upload files to its file system without sufficiently validating things like their name, type contents or size. Failing to properly enforce restrictions on these could mean that even a basic image upload function can be used to upload arbitrary and potentially dangerous files instead. This could even include server-side script files that enable remote code execution.



# Impact of file upload vulnerabilities

The impact depends on two key factors:

- Which aspect of the file the website fails to validate properly.

- What restrictions are imposed on the file once it has been successfully uploaded.

In the worst case scenario, if the file's type isnt validated properly, and the server configurations allows certain types of file (such as .php and .jsp) to be executed as code. In this case, an attacker could potentially upload a server-side code file that functions as a web shell, effectively granting them full control over the server.

If the filename isn't validated properly, this could allow an attacker to overwrite critical files simply by uploading a file with the same name. If the server is also vulnerable to directory traversal, this could mean attackers are even able to upload files to unanticipaed locations.

Failing to make sure that the size of the file falls within expected thresholds could also enable a form of Dos attack.

# How do file upload vulnerabilities arise ?

Given the fairly obvious dangers, its rare for websites in the wild to have no restrictions on which files are allowed to be uploaded. More commonly developers implement what they believe to be robust validation that is either inherently flawed or can be easily bypassed.

Example: Devs may attempt to blacklist dangerous file types but fail to account for parsing discrepancies when checking the file extensions. As with any blacklist, it's also easy to accidentally omit more obscure file types that may still be dangerous.

In other cases, the website may attempt to check the file type by verifying properties that can be easily manipulated by an attacker using tools like Burp Proxy or Repeater.

Even robsst validation measures may be applied inconsistently across the network of hosts and directories that form the website, resulting in discrepancies that can be exploited.

# How do web servers handle requests for static files ?

At some point, the server parses the path in the req to identify the file extension. It then uses this to determine the type of the file being requested, typically by comparing it to a list of preconfigured mappings between extensions and MIME types. What happens next depends on the file type and the servers congfiguration:

- If this file type is non-executable, such as image or static HTML page, the server may just send the contents to the client in HTTP response
- If the file is exe, such as a PHP file and the server is configured to execute files of this type, it will assign variables based on the headers and parameters in the HTTP request before running the script. The resulting output 


**The content-type response header may provide clues as to what kind of file the server thinks it has served**

# Exploiting unrestricted file uploads to deploy a web shell

From a security perspective, the worst possible scenario is when a wesbite allows to uupload server-side scripts, such as PHP, java, or python files and is also configured to execute them as code. 

Web shell: A web shell is a malicious script that enables an attacker to execute arbitrary commands on a remote web server simply by sending HTTP requests to the right endpoint.

If successfully able to upload a web shell, we will have full control over the server. This means that we can read or write arbitrary files, exfiltrate sensitive data, even use the server to pivot attacks against both internal infrastructure and other servers outside the network

### Lab: Remote code execution via web shell upload

This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter

1. First we log in and we can see that theres an option to attache a file.

![alt text](image-238.png)

2. Lets upload a random image and see what happens.

![alt text](image-239.png)

![alt text](image-240.png)

![alt text](image-241.png)

The random screenshot has been uploaded.

3. Lets have a look at the burps history. Here we can see that a lot of requests are being made. Lets filter type by MIME and enable the images checkbox and apply the changes. But before that lets send the POST request to repeater.

![alt text](image-242.png)


# Exploiting flawed validation of the uploads

## Flawed file type validatoin

When submitting HTML forms, the browser typically sends the provided data in a POST request with the content type application/x-www-form-urlencoded. This is fine for sending simple text like name or address. However, it isn't suitable for sending large amounts of binary data, such as entire image file or a PDF document. In this case, the content type multipart/form-data is preferred.


### Lab: Web shell upload via Content-Type restriction bypass

This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter

1. As usual, lets login to the account and upload an image.

![alt text](image-243.png)

### Lab: Preventing file execution in user-accessible directories

While its better to prevent dangerous file types being uploaded in the first place, the second line of defense is to stop the server from executing any scripts that do slip through the net.

Servers generally only run scripts whose MIME type they have been explicitly configured to execute. Otherwise, they might just return some kind of error message or, in some cases, serve the contents of the file as plain text instead:

This attempt might leak the source code but will nullify any attempt to create a web shell

This kind of configuration often differs between directories. A directory to which user-supplied files are uploaded will likely have much stricter controls than other locations on the filesystem that are assumed to be out of reach for end users. If an attacker can find a way to upload a cript to a different directory thats not supposed to contain user-supplied files, the server may execute your script after all.

Even if we send all of our requests to the same domain name, this often points to a reverse proxy server of some kind such as a load balancer which can also be configured differently.

## Insufficient blakclisting of dangerous file types

One of the more obvious ways of preventing users from uploading malicious scripts is to blacklist potentially dangerous file extension like ".php". The practice of blacklist is flawed as it's difficult to explicitly block every possible file extension that could be used to execute code. Such blacklists can sometimes be bypassed by using lesser known, alternative file extensions that can still be executable such as .php5, .shtml and so on.

### Overriding the server configuration

As dicussed in the prev section, servers typically wont execute files unless they have been configured to do so. For example, before an Apahce server will execute PHP files requested by a client.

Many servers also allow developers to create special config files within individual directories in order to overrride or add to one or more of the global settings. Apache servers, for example, will load a directory-specific configuration from a file called ".htaccess" if one is present. 

Similarly, developers can make directory-specific configuration on IIS servers using a web.config file. This might include directives such as the following, which in this ase allows JSON files to be served to users.

Web servers use these kinds of configuration files when present, but not normally allowed to access using HTTP requests. Occasionally finding servers that fail to stop from uploading your own malicious configuration file. 

### Obfucating file extensions

Blacklists can be potentially be bypassed using classic obsfucation techniques. If the validation code is case sensitive and fails to recognize that exploit.pHp is a .php file then it can be bypassed easily. If the code that subsequently maps the file extension to a MIME type is not case sensitive, this discrepancy allows to sneak malicious PHP past validation that may eventually be executed by the server. 

Other techniques of achieving similar results are:

- Provide multiple extensions. Depending on the algorithm used to parse the filename, the following may be interpreted as either a PHP file or a JPG image: exploit.php,jpg

- Adding trailing characters: some component will strip or ignore trailing whitespaces, dots.

- Using URL encoding fordots, forward slashes, and backward slashes. If th values isnt decoded when validating the file extension, but is later decoded server-side, this can also allow to upload malicious files that could otherwise be blocked.

- Adding semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in c/c++, example,

Other defenses involve stripping or replacing dangerous extensions to prevent the file from being executed. If the transformtaion isn't applied recursively,