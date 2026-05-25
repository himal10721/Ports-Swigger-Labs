# Business Logic Vulnerabilties

The vulnerabitlies in the design and implementation of an application that can allow an attacker to elicit unintended behavior. This enables the attackers to manipulate legitimate functionality to achieve malicious goals. 

In simple words, the attackers might be able to do some things that they shouldn't be allowed in the first place. For an example, they might be able to complete a transaction without going through the purchase workflow.

Logic-based vulns can be diverse and are often unique to the application and its specific functionality. This makes it difficult to detect using automated vulnerability scanners. 

Business logic vulnerabilties often arise because of the design and developmenet teams mistakes.
In order to avoid logic flaws, developers need to understand the applicaction as a whole. Developers working on large code bases might not have a proper understanding of how all areas of the application work. If the devs donot expilictly document any assumptions that are being made, it is easy for these kinds of vulnerabilties to creep into an application.


## Excessive trust in client-side controls

### Lab: Excessive trust in client-side controls

1. First we try to buy the leather jacket but get rejected becuase there isn't enough store credit.
   
![alt text](../images/image-128.png)

2. We now need to study the process of purchase.

![alt text](../images/image-129.png)

Here we can see that there is a POST request being made to /cart after the item has been added to cart. 

3. We send this to burp repeater and change the value to 100 and send the request.
![alt text](../images/image-130.png)

4. In the end we purchase it to solve the labs.

![alt text](../images/image-131.png)


### Lab 3: 2FA Broken logic

1. First we check through the 2FA process and see how it works.

2. Here we can see a few get and post request being made to the server.
![alt text](../images/image-148.png)

3. The GET /login2 uses the verify parameter to check to confirm the user, we send this to repeater to see if it works and sends us to the mfa code page or not. 

![alt text](../images/image-132.png)

4. We can see that the GET /login2 sends us directly towards the 4-digit security code page. So this means that a temporary 2FA code was generated for carlos.

5. The POST/ login 2 sends the mfa-code. 

![alt text](../images/image-133.png)

6. We now login again with wiener and peter. We submit an invalid code. We send the request to intruder and set the parameter verify to carlos and add a payload position of mfa-code and brute force the payload. But since im running burpsuite community edition, it will be impossible to do that. 

## Failing to handle unconventional input

One aim of the application logic is to restrict user input to values that adhere to the business rules.
For an example, When ordering any products, users typically specify the quantity that they want to order. Although any integer is theoretically a valid input, the business logic might prevent users from ordering more units that are currently in stock.

### Lab: High-level logic vulnerabiltiy

1. We first normally tey to purchase an item and see the process of purchase in burpsuite.


2. Here we can see that a POST request is being made to /cart. We change the quantity to a negative number and send it there. 

![alt text](../images/image-134.png)

3. We can now see that the total price has changed to -1337.00. 
![alt text](../images/image-135.png)

4. We now add another item to bring back the price to a positive value just wihtin our $100 store credit.

![alt text](../images/image-136.png)

Here, by carefully calculating everything, we sucessfully purchased a lightweight leather jacket worht $1337.00

### Lab: Low-Level logic flaw:

This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter

1. As usual, first we identify the workflow of the purchase. 

![alt text](../images/image-137.png)

The process is similar to the one we just did!

2. We can only add 2 digit quantity. If we add 3 digit number like "100", then we get an error message.
![alt text](../images/image-138.png)

![alt text](../images/image-139.png)

3. So we now need to go to intruder and set the quantity parameter to 99. And configure the payload type as Null payloads and payload configuration as continue indefinitely.

![alt text](../images/image-140.png)

We now start the attack.

4. Now we can see that there's a lot of quantity being added. We want the application to crash or to behave in an abnormal way.

![alt text](../images/image-141.png)


![alt text](../images/image-142.png)

So at one point the total amount should equal to negative due to wraparound. Since the maximum value is not assigned, after the max value assigned it goes to the smallest integer. This is a typical behavior from a signed integer. 
Then we need to carefully calculate and start adding items in reverse order to balance out the credit as we want it to be under $100.00.
My version of burp suite is community so It will take me alot of time to complete this lab.. :)

### Lab: Inconsistent handling of exceptional input

1. We check the behavior of the application registration process. 

![alt text](../images/image-143.png)
   
2. We try to truncate characters and see what happens. So we first need to send the POST /register request to intruder and add the email prefix as payload.

![alt text](../images/image-144.png)

3. In payload type, we need to select character blocks and configuration is minimum length 100 and maximum length 1000.

![alt text](../images/image-145.png)

4. We can see that there is a list of AAAAAAAAA..........., and no domain shown. We can now carefully craft a payload which will read till @dontwannacry only.

![alt text](../images/image-146.png)

5. We now need to create a new user and capture the POST /register request and manipulate the email parameter there. 


6. Now we can see that we get HTTP 200 response, so everything is good to go.



## Trusted users wont always remain trustworthy

Applications may appear to be secure because they implement seemingly robust measures to enforce the business rules. Unfortunately, some applications make the mistake of assuming that, having passed these strict controls initially, the user and their data can be trusted indefinitely.
If business rules and security measures are not applied consistently throughout the application, this can potentially lead to dangerous loopholes that may be exploited by an attacker.

This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. To solve the lab, access the admin panel and delete the user carlos.

1. First lets look at the /admin path and see if we're allowed
![alt text](../images/image-150.png)

It will only allows us if we are logged in as DontWannaCry user.

So let's go ahead and create an account

2. Lets login and try to change the email address to @dontwannacry
![alt text](../images/image-151.png)


3. Surprisingly, our email has been changed and we can now access the Admin Panel
![alt text](../images/image-152.png)

4. We delete the user "carlos" and solve the lab!
![alt text](../images/image-153.png)

![alt text](../images/image-154.png)

## Users always wont follow the intended sequence

### Lab: Insufficient workflow validation

This lab makes flawed assumptions about the sequence of events in the purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

1. Lets log in to our account using peter and wiener.

2. Lets observe the purchase workflow. 
![alt text](image.png)

We can see that there are a few POST request and GET requests being made.

![alt text](image-1.png)

3. We send the POST /cart to repeater and lets paly around with the parameters and see if it works or no

4. Lets send the /cart/order-confirmation?order.... to repeater and add the leather jacket to basket to see if the program actually checks the cart.
We simply need to send this request to the server
![alt text](image-5.png)

5. We solve the lab!
![alt text](../images/image-155.png)

## Lab: Authentication bypass via flawed state machine


This lab makes flawed assumptions about the sequence of events in the login process. To solve the lab, exploit this flaw to bypass the lab's authentication, access the admin interface, and delete the user carlos.

You can log in to your own account using the following credentials: wiener:peter

1. We login and try to access the dmin page by entering /admin in the url.

![alt text](../images/image-160.png)

But it didnt work!

2. Lets review the Burp process and see what's going on.

![alt text](../images/image-162.png)

we can see that at POST /role-selector we can select the role. Let's play round with the role parameter.

3. Lets send the request to burp repeater and set the role to admin.

![alt text](../images/image-163.png)


didnt work. Lets turn on the intercepter and see if we can do it from there.

4. After selecting the POST request, lets change the role to admin and forward it
![alt text](../image/image-164.png)

5. After thats done now lets try to access the admin page and see if it works. 

6. It didnt work, so now lets try to drop the packet, access the home page and see if that works.
   
![alt text](../image-166.png)

7. We can see that we now have access to admin panel page!

![alt text](../images/image-167.png)

8. Lets delete carlos user and solve the lab!
![alt text](../images/image-168.png)

## Domain Specific flaws

The discounting functionality of online shops is a classic attack surface when hunting for logic flaws. Example, an online shop that offers 10% discount on orders over $1000. This is vulnerable if the system fails to validate if the order was changed after the discount is applied. In this case, the attacker could simple add items to their cart until they hit the $1000 threshold, then remove the items they dont want before placing the order.

### Lab: Flawed enforcement of business rules

LAB
Not solved
This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter

1. Lets check the dicount functionality first.
![alt text](../images/image-170.png)
![alt text](../images/image-171.png)

Theres a post request being made to /cart/coupon

2. Lets send this to repeater and send the request again to see if we can further apply the coupon code!

![alt text](../images/image-172.png)

We can see that it rejects the new code. 
3. Lets alternate between the two coupons and see if it lets us by pass this!
![alt text](../images/image-173.png)

4. We can see that it does indeed let us add as much coupon as possible. Lets alternate between the coupons and purchase it !

![alt text](../images/image-174.png)

AFter alternating the coupon for a few times we can now get this jacket for 0 dollars!

5. We place order in order to complete the lab!
![alt text](../images/image-175.png)


### Lab: Infinite money logic flaw

This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter

1. first lets log in to our account and try purchasing the item in order to investigate the flow of the program

2. There a singup to newsletter, let go a head and sign up and see if we receive any codes or coupons
![alt text](../images/image-176.png)

There is indeed a coupon. lets apply the coupon and see how much discount we can get!
![alt text](../images/image-177.png)

We get $401.10 off on discounts
![alt text](../images/image-178.png)

3. We purchase the gift card for $10 and lets try to aapply the gift card code in there!
![alt text](../images/image-179.png)

4. We can see the POST request to /gift-card
![alt text](../images/image-180.png)

1. We now need to go to settings and in the session handling rules panel, we click add. The session handling rule edito dialog opens.

![alt text](../images/image-181.png)

6. We select the following sequence of request in the macro recorder

![alt text](../images/image-182.png)

7. We configure this item, add to create a custom parameter. We name the parameter gift-card and highlight the gift card code at the bottom of the response. ALso add a parameter handling to response 4.

![alt text](../images/image-183.png)

8. We test the macro. We can see in the screenshot below that the macro generated a new giftcard.

![alt text](../images/image-184.png)

9. We send the request GET /my0account to intruder!
![alt text](../images/image-185.png)



## Providing an encryption oracle

It is dangerous when user-controllable input is encrypted and the resulting cipgertext is then made available to the user in some way. This kind of input is sometimes known as "encrpytion oracle". An attacker might use this input to encrpyt arbitrary data using the correct algorithm and asymmetric key. 

This becomes dangerous when there are other user-controlled inputs in the application that expect data encrpyted with the same algorithm. In this case, an attacker could potentially use the encryption oracle to generate valid, encrpyted input and then pass it into other sensitive functions.

This issue can be compounded if there is another user-controllable input on the site that provides the reverse funciton. This would enable the attacker to decrypt the data to indentify the expected structure. This saves them some of the work involved in creating their malicious data but is not necessarily required to craft a successful exploit.

## Email address parser discrepancies

Some websites parse email addresses to extract the domain and determine which organization the email owner belongs to. while this process may initially seem straightforward, it is actually very conplex, even for valid RFC-compliant addresses.

Discrepancies in how email addresses are parsed can undermine this logic. These discrepancies arise when different parts of the application handle email addressses differently.

Discrepancies in how email addresses are parsed can undermine this logic. These discrepancies arise when different parts of the application handle email addresses differently.

An attacker might exploit these discrepancies using encoding techniques to disguise parts of the email address. This enables the attacker to create email addresses that pass initial validation checks but are interpreted differently by the server's parsing logic.

The main impact of email address parser discrepancies is unauthorized access. Attackers can register accounts using seemingly valid email addresses from restricted domains. This enables them to gain access to sensitive areas of the application, such as admin panels or restricted user functions.

