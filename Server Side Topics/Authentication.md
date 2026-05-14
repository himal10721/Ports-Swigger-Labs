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
