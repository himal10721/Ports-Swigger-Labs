# Path traversal

Also known as directory traversal. This vuln enables an attacker to read arbitrary files on the server that is running an application.

This can include:

Application and code data
Creds for backend systems
Sensitive operating system files

# Reading arbitrary files via path traversal

### Lab: File path traversal, simple case

This lab contains a path traversal vulnerability in the display of product images.

To solve the lab, retrieve the contents of the /etc/passwd file.

1. Lets send the GET request to repeater and play around with it.
![alt text](../images/image-217.png)

2. We change the file path to a different one and see if we get any response.
   
![alt text](../images/image-218.png)

3. We can see the HTTP 200 ok request, that means it is valid and we are good to go.

![alt text](../images/image-219.png)

4. Now, lets try to go one directory back and see what happens.

![alt text](../images/image-220.png)

We try to get out of the directory and we got error.
![alt text](../images/image-221.png)
So lets try tp travel even further behind and see what happens.

5. We can see that we got an ok message, so that mean we have found the file.

![alt text](../images/image-222.png)  

This is all the secret files that was on the server!

![alt text](../images/image-223.png)


# Common obstacles to exploiting path traversal vulnerabilities

If an application strips or blocks directory traversal sequences from the user-supplied filename, it can be possible to bypass the defense using a variety of techniques.

Absolute path from the filesystem root can be used such as filename=/etc/passwd, to directly reference a file without using any traversal sequences.

### Lab: Lab: File path traversal, traversal sequences blocked with absolute path bypass

This lab contains a path traversal vulnerability in the display of product images.

The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.

To solve the lab, retrieve the contents of the /etc/passwd file.

1. We get a GET request, requesting images from the server. Lets send this to repeater by pressing ctrl+r.

![alt text](../images/image-224.png)

2. We can see the path the image is coming from, lets send a get request to that path..

![alt text](../images/image-225.png)

3. We get a 200 ok response!

![alt text](../images/image-226.png)

4. By simply changing the file path to absolute path, we can see that we have successfully retrieved all the files.

![alt text](../images/image-227.png)

### Lab: File path traversal, traversal sequences stripped non-recursively

This lab contains a path traversal vulnerability in the display of product images.

The application strips path traversal sequences from the user-supplied filename before using it.

To solve the lab, retrieve the contents of the /etc/passwd file.

1. As usual, we send the get request to repeater.

![alt text](../images/image-228.png)

2. Lets change the path to perform a path traversal.

![alt text](../images/image-229.png)

3. Lab solved :)


## Important: In some cases, such as a URL path or the **Filename** parameter of a multipart/form-data request, web servers may strip any direcotry traversal sequences before passing your input to he application. These can be bypassed by using URL encoding, or even double URL encoding, the **../** characters.  

### Lab: File path traversal, traversal sequences stripped with superfluous URL-decode

This lab contains a path traversal vulnerability in the display of product images.

The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.

To solve the lab, retrieve the contents of the /etc/passwd file.

1. As before, lets send this GET request to repeater.

![alt text](../images/image-230.png)

2. Lets try to play around with the source of the image and try to perform a path traversal.

3. Checking with absolute path, it failed..

![alt text](../images/image-231.png)

Before we move forward lets try to URL encode the "/" and see if we get any response.

![alt text](../images/image-232.png)

As we can see that there is no path found even after URL encoding the "/" in the parameter.

4. Lets try a different format, "../../etc/passwd"

![alt text](../images/image-233.png)

5. Lets URL encode the "/"

![alt text](../images/image-234.png)

As we can see that we have solved the lab after successfully encoding the URL.

:)))


### Lab: File path traversal, validation of start of path

This lab contains a path traversal vulnerability in the display of product images.

The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.

To solve the lab, retrieve the contents of the /etc/passwd file.

1. Lets send the get request of the image to repeater and play around with it.

2. By simply appending the ../ x3 times, we can easily access the password directory.

![alt text](../images/image-235.png)

### Lab: File path traversal, validation of file extension with null byte bypass

This lab contains a path traversal vulnerability in the display of product images.

The application validates that the supplied filename ends with the expected file extension.

To solve the lab, retrieve the contents of the /etc/passwd fil

1. Lets send the request to repeater and see change the GET request 
![alt text](../images/image-236.png)

In the below window, we can see that there is a 200 OK message, so we're good to go!!

2. Lets replace the file path to: ../../../etc/passwd%00.png

![alt text](../images/image-237.png)

We can see that the lab is successfully solved!!

