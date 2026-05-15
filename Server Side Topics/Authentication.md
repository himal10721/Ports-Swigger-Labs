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
     
   - We successfully login and solve the lab.## Lab: Broken Brute-Force Protection, IP Block

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