# Authentication Vulnerabilities

Authentication vulnerabilities allow attackers to gain access to sensitive data and functionality. For this reason, it is important to learn how to identify and exploit authentication vulnerabilities and understand how to bypass them.

---

## What is Authentication?

Authentication is the process of verifying the identity of a user or client. The three main types of authentication are as follows:

1. **Something you know**: A password or the answer to a security question.
2. **Something you have**: A physical card or a token.
3. **Something you are**: Biometrics or patterns of behavior.

---

## Difference Between Authentication and Authorization

- **Authentication**: Determines whether a user is who they claim to be.
- **Authorization**: Ensures that the user, once verified, is allowed to perform specific activities.

---

## Vulnerabilities in Password-Based Login

### Brute-Force Attacks

A brute-force attack occurs when an attacker uses a system of trial and error to guess valid user credentials. These attacks are typically automated using wordlists of usernames and passwords. Attackers can also use publicly available knowledge to fine-tune brute-force attacks and increase the chance of successfully bypassing password checks.

#### Brute-Forcing Usernames

Usernames are easier to guess if a regular pattern is confirmed, like an email address. While auditing, check whether the website discloses potential usernames publicly.

#### Brute-Forcing Passwords

Passwords can be brute-forced, with the difficulty varying based on the strength of the password. Many websites adopt password policies that force users to create high-entropy (random) passwords that are harder to crack using simple brute-force methods. However, users might create predictable variations, such as:

- If "mypassword" is not allowed, users might try:
  - `Mypassword!`
  - `mypas$$word`

---

### Username Enumeration

When an attacker observes changes in a website's behavior to identify whether a given username is valid, it is called username enumeration. This reduces the time and effort required to brute-force a login because the attacker can quickly generate a shortlist of valid usernames.

#### Indicators of Username Enumeration

While attempting brute-force attacks on a login page, pay attention to any differences in:

1. **Status codes**
2. **Error messages**
3. **Response times**

---

## Lab: Username Enumeration via Different Responses

### Steps:

1. **Submit Invalid Credentials**
   - Submit an invalid username and password.
   - Highlight the username parameter in the `POST /login` request and send it to the Intruder.

   ![Invalid Credentials](../images/image-11.png)

2. **Configure Grep-Extract**
   - Go to settings and under Grep-Extract, add the following:

     ![Grep-Extract Settings](../images/image-12.png)

   - Highlight the part "Invalid username or Password" and press OK. Everything else is handled automatically.

     ![Highlight Error Message](../images/image-13.png)

3. **Start the Attack**
   - Start the attack. After getting the results, observe the new column named `-warning>`.

     ![Attack Results](../images/image-14.png)

4. **Analyze Results**
   - Notice that all error messages are the same except one, which is different (e.g., missing a full stop).
   - Use this to identify the valid username.

     ![Analyze Results](../images/image-16.png)

5. **Brute-Force Passwords**
   - After identifying the username, repeat the process for the password.
   - Observe the different status code (e.g., `302`) for a payload named "amanda".

     ![Password Brute-Force](../images/image-17.png)

6. **Login and Solve the Lab**
   - Use the identified username and password to log in.

     - Username: `asdl`
     - Password: `amanda`

     ![Login Success](../images/image-18.png)

   - After logging in, the lab is solved!

     ![Lab Solved](../images/image-19.png)

---

## Lab: Username Enumeration via Response Timing

### Steps:

1. **Capture the Login Request**
   - With Burp Suite running, enter an invalid username and password.
   - Send the `POST /login` request to Burp Repeater.

2. **Spoof IP Address**
   - To bypass IP-based brute-force protection, add the `X-Forwarded-For` header.
   - Change the code from `500` to a different number (e.g., `501`, `502`) for each request.

     ![Spoof IP Address](../images/image-20.png)

3. **Analyze Response Times**
   - Observe the response times:
     - Shorter response time: Correct username and password.
     - Slightly longer response time: Valid username but incorrect password.

     ![Response Time Analysis](../images/image-21.png)
     ![Response Time Comparison](../images/image-22.png)

4. **Configure Burp Intruder**
   - Send the request to Burp Intruder.
   - Select the Pitchfork attack type and add the `X-Forwarded-For` header.
   - Add payload positions for both `X-Forwarded-For` and the username.

     ![Burp Intruder Setup](../images/image-23.png)

5. **Set Payloads**
   - Set the password payload to a long string (e.g., 100 characters).
   - In the payloads panel, configure:
     - Payload position: `1`
     - Range: `1-100`
     - Step: `1`
     - Max fraction digits: `0`

     ![Payload Configuration](../images/image-24.png)
     ![Payload Range](../images/image-25.png)

6. **Start the Attack**
   - Start the attack and tick the checkbox for "Response Completed" in the results.

     ![Start Attack](../images/image-26.png)
     ![Response Completed](../images/image-27.png)

7. **Identify Valid Username**
   - Analyze the results to find significant differences in response times.
   - Confirm the valid username by repeating the request multiple times.

     ![Identify Username](../images/image-28.png)
     ![Confirm Username](../images/image-29.png)

8. **Brute-Force Passwords**
   - Create a new Burp Intruder attack.
   - Add the `X-Forwarded-For` header and set the payload position to the password parameter.
   - Use the identified username (e.g., `adm`) and paste the provided password list.

     ![Brute-Force Passwords](../images/image-31.png)

9. **Login and Solve the Lab**
   - After identifying the correct password (e.g., `killer`), log in using:
     - Username: `adm`
     - Password: `killer`

     ![Login Success](../images/image-32.png)
     ![Lab Solved](../images/image-33.png)


## Lab: Broken Brute-Force Protection, IP Block

### Steps:

1. **Investigate the Login Page**
   - We first investigate the login page and observe that the IP is temporarily blocked if we submit 3 incorrect logins in a row.
   - However, we can reset the counter by logging in to our own account before this limit is reached.
   
     ![alt text](../images/image.png)

2. **Configure Burp Intruder**
   - We need to enter an invalid username and password and then send the `POST /login` request to the Burp Intruder.
   - We then create a Pitchfork attack with a payload position on both the username and password parameters.
   
     ![alt text](../images/image-101.png)

3. **Set Up Resource Pool**
   - We click on the resource pool to open the Resource pool side panel, then add the attack to a resource pool with maximum concurrent requests set to `1`.
   - By only sending one request at a time, we can ensure that our login attempts are sent to the server in a correct order.
   
     ![alt text](../images/image-102.png)

4. **Configure Payloads (Position 1)**
   - After this, we click on payloads to open the Payloads side panel and select position `1` from the payload position drop-down list.
   - We add a list of payloads that alternates between our username and `carlos`.
   - Also need to make sure that our username is first and `carlos` is repeated at least 100 times. For this, we run a script from GitHub by rkhal101.
   
     ![alt text](../images/image-103.png)
     ![alt text](../images/image-104.png)

5. **Configure Payloads (Position 2)**
   - Editing the list of candidate passwords is what we do now.
   - We need to add our own password before each one and need to make sure that the password is aligned with the username in the other list.
   
     ![alt text](../images/image-105.png)
     
   - We select position `2` from the Payload position drop-down list and add the password list. After that we start the attack.

6. **Analyze Results**
   - When the attack finishes, we filter the results to hide the responses with a `200` status code and sort out the remaining results by username.
   - There should only be a single `302` response for requests with the username `carlos`. Also need to make a note of the password from the Payload 2 column.
   
     ![alt text](../images/image-106.png)
     
   - *In the above screenshot, we can see that the username `carlos` matches with password `131313`.*

7. **Login and Solve the Lab**
   - At last, we login into Carlos's account using the password we identified and access his account page.
   
     ![alt text](../images/image-107.png)

---

## Account Locking

One way in which websites try to prevent brute-forcing is when certain criteria are met, usually a set number of failed login attempts. Just as with normal login errors, responses from the server indicating that an account is locked can also help us enumerate usernames.

---

## Lab: Username Enumeration via Account Lock

### Steps:

1. **Configure Cluster Bomb Attack**
   - We investigate the login page in Burp by sending the `POST /login` request to Burp Intruder.
   - We then select a **Cluster bomb** attack from the attack type drop-down menu and add the `username` parameter to the payload position and also add a blank payload position to the end of the request body.
   
     ![alt text](../images/image-108.png)

2. **Set Payloads**
   - In the payloads panel, we add the list of usernames for the first payload position from PortSwigger.
   - In the second payload position which is empty, we select the **Null payloads** type and choose the option to generate `5` payloads which will cause each username to be repeated 5 times, and start the attack.
   
     ![alt text](../images/image-109.png)

3. **Analyze Results**
   - In the results, we can notice that the responses for one of the usernames are longer than the responses when using other usernames.
   - One username has a different error message, we make a note of this username.
   
     ![alt text](../images/image-110.png)

4. **Brute-Force Passwords**
   - We now create a new Burp Intruder attack on the `POST /login` request but select **Sniper** attack this time.
   - We set the `username` parameter to the username that we identified and add a payload position to the `password` parameter and paste the list of password parameters.
   - Once this is done, we simply start the attack.
   
     ![alt text](../images/image-111.png)

5. **Login and Solve the Lab**
   - We look for the one that did not have any error messages. In our case, it's "cheese".
   - We make a note of this and go to the login page.
   
     ![alt text](../images/image-112.png)
     
   - We successfully login and solve the lab.



### User Rate Limiting

User rate limiting can prevent brute-force attacks. Making too many requests within a short period of time causes the IP address to be blocked.

#### Lab: Broken brute-force protection, multiple credentials per request

1. **Investigate the Login Page**
   - The `POST /login` request submits the login credentials in JSON format. Send this request to the repeater.
     ![alt text](../images/image-38.png)

2. **Modify the Password Field**
   - Replace the single string value of the password with an array of strings containing all the candidate passwords.
     ![alt text](../images/image-34.png)

3. **Send the Request**
   - Send the modified request. This returns a `302` response.
     ![alt text](../images/image-35.png)

4. **Show Response in Browser**
   - Right-click on the request and select "Show response in browser."
   - Copy the URL and load it in the browser. The page loads, and you are logged in as `carlos`.
     ![alt text](../images/image-37.png)
     ![alt text](../images/image-36.png)
     
# Vulnerabilities in Multi-Factor Authentication

## Lab: 2FA Simple Bypass

1. **Log In to Own Account**
   We log into our own account. The 2FA will be sent by email.

2. **Note the Account URL**
   We go to our accounts page and make a note of the URL.
   ![alt text](../images/image-45.png)

3. **Log Out**
   We log out of our account.

4. **Log In with Victim's Credentials**
   We log in using the victims credentials.

5. **Bypass Verification**
   When prompted for the verification code, we change the URL to navigate to `/my-account`.
   ![alt text](../images/image-39.png)

---

## Lab: Flawed Two-Factor Verification Logic

1. **Investigate the Verification Process**
   We log in to our own account and investigate the 2FA verification process. We use `POST /login2` request and verify the parameter that is being used to determine which user's account is being accessed.
   ![alt text](../images/image-40.png)

2. **Log Out**
   We log out of our account.

3. **Generate Temporary 2FA Code**
   We send the `GET /login2` request to burp repeater and change the value of the `verify` parameter to `carlos` and send the request to ensure a temporary 2FA code for carlos.
   ![alt text](../images/image-41.png)

4. **Submit Invalid Code**
   We go to the login page and enter the username and password and submit an invalid 2FA code.
   ![alt text](../images/image-42.png)

5. **Send Request to Intruder**
   We then send the `POST /login2` request to Burp intruder.
   ![alt text](../images/image-43.png)

6. **Brute-Force the Code**
   In burp intruder, we set the `verify` parameter to `carlos` and add the payload position to the `mfa-code` parameter and brute-force the code.
   ![alt text](../images/image-44.png)

7. **Solve the Lab**
   We then load the `302` response in the browser and click my account to solve the lab.

---

# Lab 2FA: bypass using a brute-force attack

With passwords, websites need to take steps to prevent brute-forcing of the 2FA verification code. This is important because the code is often a simple 4 or 6 digit number.

1. we log in as carlos which is already provides and investigate the 2FA process. If we enter the wrong code twice, we will be logged out again. We need to use Burp's session handling features to log back in automatically before sending each request.


2. We got to settings to open the settings dialog and click sessions. 

3. In the dialog, we go to the scope tab and unser the URL scope, we select the option to include all the URLs.



4. We go back to the Details tab and under Rule Actions, we click Add and then Run a macro.

5. We select macro and click add to open the macro reader.
![alt text](../images/image-46.png)

6. We now send /login2 and post/login2 to the intruder and add the payload positions. We also set the payload type to Brute forcer and set character set to 0123456789 so that only integers are brute forced.
![alt text](../images/image-47.png)

7. Since im using the brup community edition, i need to wait for a very long time. So, i cannot complete the lab.

# Vulnerabilities in other authentication mechanisms:

In addition to the login functionality, websites provide another functionality which allows users to manage their account. Users can change their password or reset their password when they forget it. The mechanisms can also introduce vulnerabilities that can be exploited by an attacker. Websites usually take care to avoid well-known vulnerabilities in their login pages. It is easy to overlook the fact that you need to take similar steps to ensure that related functionality is equally as robust. 

## Keeping users logged in:
Another feature is the option to stay logged in even after closing a browser session. This means checking a simple checkbox labeled something such as "Remember me" or "Keep me logged in".

The functionality is often implemented by generating a "remember ne" token of some kine, which is then stored in a persistent cookie. This cookie effectively allows to bypass the entire login process and is the best practice for the cookie to be impractical to guess. However, some websites generate the cookie based on predictable concatentation of static values such as username anda a timestamp. Some even use the password as part of the cookie. This is risky because the attacker can create their own account and can study their own cookie and deduce how it is generated. Once they work out the formula, they can try to brute-force other users' cookies to gain access to their accounts.

Some websties evem assume that if the cookie is encrypted in some way, it will not be guessable even if it does use static values. While this may be tru if done correctly, naively "encrpyting" the cookie using a simple two-way encoding with a one-way hash function is not completely bulletproof. If the attacker is able to easily identify the hashing algorithm, and no salting is used, they can potentially brute-force the cookie by simplky hashing their wordlists. This method can be used to bypass login attempt limits if a similar limit isn't applied to cookie guesses.

Even if the attacker is not able to create their own account, they may still be able to exploit this vulnerability using techniques, such as XSS, an attacker could steal another user's "remember me" cookie and deduce how the cookie is constructed from that. If the website was bult using an open-source framework, the key details of the cookie construction might be publicly documented.

In some cases, the attacker might be able to read the user's actual password in cleartext from a cookie. 

## LAB: Brute-forcing a stay-logged-in cookie

The lab allows users to stay logged in even sfter they close their browser sessions. THe cookie used to provide this functionality is vulnerable to brute-forcing.

1. We log into our own account with the Stay logged in option selected. This sets a stay-logged-in cookie.

![alt text](../images/image-48.png)
![alt text](image-49.png)

2. In the inspector panel, we can see that ther eare two different types of cookies, the first one allows us to stay logged in even after closing the browser.
![alt text](../images/image-50.png)

3. we go to decoder and decode the cookie. We can see the plaintext below:
![alt text](../images/image-51.png)

Given that the plaintext is the username, there is a possibility that the hashed password could be a hash of the password. So the cookie is contructed as follows:

base64(username+ ':' +md5HashofPassword)
Our taget is to get access to carlos's account. So carlos's account also mus tfollow the same cookie structure. 
Hence, it will be: base64(username + ':' + every posibility of md5 hash)


4. We log out of the account.

5. We then send the POST request where the stay logged in cookie is and clear the session because if we provide an invalid stay-logged-in cookie then it will automatically generate a new session. 
![alt text](../images/image-52.png)

After this, we add the stay-logged-in cookie in the payloads and payload type is simple. We will paste the list provided.

But we simply cant paste plaintexts in the payload since the cookie format is base64(username+ ':' + md5HashofPassword). For that we make the use of payload processing. 
In the paylaod processing, we add the md5 hash, add prefix as carlos.
![alt text](../images/image-53.png)

We also add the encode, base64-encode. 
![alt text](../images/image-54.png)


It should look like this:
![alt text](../images/image-55.png)

This is necessary because for each payload, it will first MD5 the hash, add prefix carlos, and encode the base64.

6. We simply start the attack after this. We should get a status code of 200. In response 200, we can see that we're logged in as carlos.
![alt text](../images/image-56.png)

## Lab: Offline password cracking:

1. First we log in using a regular username and password to see how the login functionality works. We can see that in the burp proxy that it makes a post request to /login.
![alt text](../images/image-57.png) 

2. We check the stay-logged-in cookie. It looks encoded. So we copy that and paste it in the decoder in burpsuite. We can see in the screenshot below that it is indeed encode, with the cookie structure of base64(username + hash).
![alt text](../images/image-58.png)

3. Here we can see that there is no HTTPOnly flag, so any javascript in the application can access the cookie and that's what we will be doing.
![alt text](../images/image-59.png)

4. we log out and look for an XSS vulnerabiltiy. In our case we can check the comment box. We need to put in a javascript and see if it works.
![alt text](../images/image-60.png)
We put in the exploit server in the comment so that each time a user accesses the page, we get access top their cookie. 


5. We need to go to our exploit server and access the logs in there. There is a GET request from the victim containing their stay-logged-in cookie.
![alt text](../images/image-61.png)

6. We then copy the stay-logged-in cookie and put it on crackstation. BEfore that, we go to decoder and confirm. 

Here, we can see that it is indeed the credentials of carlos. The cookie structure is as same as the above one.
![alt text](../images/image-62.png)

7. We now go to creackstation to reverse the hash into plaintext.
![alt text](../images/image-63.png)

In the above screenshot, it can be seen that the password is onceuponatime.

8. With this information, we can now access carlos's account as we know his password.

9. We get access to carlos's account. Now we delete the account.

![alt text](../images/image-64.png)

We solve the lab!


## Resetting user passwords
Websites often send passwords through email. As the usual password-based authentication is impossible, websites rely on alternative methods. The password reset functionality is inherently dangerous and needs to be implemented securely.

There are a few different ways that this feature is commonly implemented, with variety of vulnerability.

Some of the ways by which websites reset their password is:

1. Sending passwords by email:
2. Resetting passwords using a URL

## Lab: Password reset broken logic:

1. First we check for the reset functionality and find out how it works.
![alt text](../images/image-65.png)

We can see that it makes a POST request to /login.

2. Since the reset link is sent through the email, we open email and click the link and we will be redirected to the reset password.
![alt text](../images/image-66.png)

In Burp suite, we can see that there is a forgot passowrd request being made, we send this to repeater.
![alt text](../images/image-67.png)

3. Upon inspecting the request, we can see that it provides a temporary reset token. 
![alt text](../images/image-68.png)

next we move on with the reset process and see what request is being made next. 

4. It makes a POST request to /forgot-password and provides a token. 
![alt text](../images/image-69.png)

5. Here, we can see that it take the token twice. We now try to manipulate it and see how it behave. For now, we change the token. 
![alt text](../images/image-70.png)
We also changed the username to carlos. And forward the reqeust to see if carlos's password was changed.

4. We getr a 302 found which is a good sign.
![alt text](../images/image-71.png)

5. Now lets go a head and see if it worked. Username= carlos and password= password.
![alt text](../images/image-72.png)

![alt text](../images/image-73.png)

We solved the lab!

## Lab: Password reset poisoning via middleware

1. First we check how the reset process works. FOr that we fist reset it and investigate on burpsuite.
![alt text](../images/image-74.png)

We can see that it performs a post request. This sends to reset link to our email.

2. After clicking on password reset link, we can see that there is another GET request being made. We send that to repeater.
![alt text](../images/image-75.png)

3. After resetting the password by clicking the link, we can see that there's been a post request made to /forgot-password/.
![alt text](../images/image-76.png)

4. We now try to perform header injection. In the header, we put in our exploit server URL.

![alt text](../images/image-77.png)

5. After this has been done, we can see that the reset link has been sent. But due to X-Forwarded-Header, the link will be sent to our server instead of the user's email.
![alt text](../images/image-78.png)

6. Here we can see that after the victim has clicked on the link, we get their cookie.
![alt text](../images/image-79.png)

7. We paste it in the GET /forgot-password.
![alt text](../images/image-80.png)

We successfully go to the reset page.

8. We copy this URL, paste it in the browser and reset carlos's password.
![alt text](../images/image-81.png)

![alt text](../images/image-82.png)
We sucessfully solve the lab!

# Changing users passwords:
Typically, changing user password invloves entering current password and then the new password twice. These pages fundamentally rely on the same process for checking that usernames and current passwords match as a normal login page does. Therefore, these pages can be vulnerable to the same technique.

Password change functionality can be particularly dangerous if it allows an attacker to access it directly without being logged in as the victim user.


## LAB: Password brute-force via password change

1. We will log in with the creds that we are already provided.

2. After logging in, we are redirected to the password change page which allows us to change our password.
![alt text](../images/image-83.png)

3. Now, we need to investigate the behavior of the password change functionality. For that, we simply put in the current password and the new password twice. 
![alt text](../images/image-84.png)

We are simply redirected to sucess page.
![alt text](../images/image-85.png)

4. Our next step is to put in the incorrect password and see what happens when trying to change the password.

When we put in the wrong password in the current password field, we get sent to the login page.
![alt text](../images/image-86.png)

5. Here, we try to put in the wrong password in purpose to see what would happen if we put incorrect password after being logged out.
![alt text](../images/image-87.png)

6. Now we need to log in again and put the current password. We put in separate passwords in both the fields and see how it behaves.
![alt text](../images/image-88.png)

We can see that when there's a wrong password, in all the fields, we get redirected to the same page.

7. Now, we send it to the repeater.
![alt text](../images/image-89.png)

8. Now, we try to send the request again by putting in the correct current password this time. But leave the pass1 and pass2 as it is. 
![alt text](../images/image-90.png)

We can see that the new passwords donot match.

![alt text](../images/image-91.png)

9. We now go to intruder and try to brute force the current password. If there's a match in current password then, we will know since the system will return new passwords donot match!
![alt text](../images/image-92.png)

10. We're looking for a different response, so we sort the list of passwords by length and see if there's any message saying new passwords donot match.
![alt text](../images/image-93.png)

Here, it can be seen that there is a message saying that the new passwords do not match.

11. To solve the lab, we simply put in the password that we just found. 

![alt text](../images/image-94.png)

