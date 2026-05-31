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

