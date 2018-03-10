---
title: "My First Pen Testing Experience"
date: 2018-03-09T18:06:13+01:00
draft: false
slug: "" 
tags: [pentest]
categories: [tutorials]
description: "When I started it all seemed so scary and difficult but the more I looked into it the easier it got ..."
showpagemeta: true
showcomments: true
---

When I started, it all seemed so scary and difficult but the more I looked into it the easier it got. For those who don't know __{{<nlink url="https://en.wikipedia.org/wiki/Penetration_test" title="penetration testing" >}}__ is the art of finding vulnerabilities in systems, networks, or web applications. Pen testers are among the best in regards to security knowledge and probably the highest paid. During my masters degree, I took a pen testing course and we were instructed to find all vulnerabilities in a given web application and write a report about it. This was one of the most exciting courses I have ever taken. My experience was amazing but I probably gave this course more time than I should have because it made me focus less on other courses I have taken and ofcourse my master thesis. The web application that we were given was in .ova format. You can find it __{{<nlink url="https://drive.google.com/open?id=1DABXCzsU9wnsXNQtNY7BPnleedAlrx0g" title="here" >}}__ if you are interested. I will ommit the full details for now but a summary of the vulnerabilities is below. 

please note that these vulnerabilities were made intentianally for educational purposes and most of them are outdated. 

The ova file contains a web application (wordpress) for cooking site called “Boutique Vegan BBQ”. The web application can be accessed by updating the hosts file (located in /etc/hosts) to include the following entry: <ip address of vm> websecurity.com.


### Vul 1: Unauthorized access to user files
2 files located at http://websecurity.com/users.txt and http://websecurity.com/test.txt contain user data available that can be read without authorization.

**Demo:**

1.	Ensure you're not logged into website

2.	Browse to either ```http://websecurity.com/users.txt``` or ```http://websecurity.com/test.txt```

![demo v101](/img/pent/v101.png)


### Vul 2: SQL Injection 1 - email
The web application is vulnerable to SQL injection. An SQL injection was found in the email field of the comment form. This allows the attacker to spoof on database information and possibly tamper with it.

**Demo:**

1.	Browse to any page you can post comments, for example ```http://websecurity.com/blog/```

2.	Use ```' OR '1'='1``` as your email address,other fields can be any random data.

3. The result of this is that it returns the first (because of LIMIT 1) user from the database which can be viewed in the page source as a comment.

![demo v201](/img/pent/v201.png)


### Vul 3: SQL Injection 2 - redirect
The application is also vulnerable to SQL injection at a different location. In the login page, the attacker can use ```redirect_to=``` in the login PHP script ```wp-login.php``` to execute any SQL queries. This attack results in the same consequences as SQL injection 1.

**Demo:**

1.	Browse to ```http://websecurity.com/wp-login.php?redirect_to=' or 1='1``` or the encoded version ```http://websecurity.com/wp-login.php?redirect_to=%27%20or%201=%271```

The result of this is that all users from the database are returned which can be viewed in the page source as a comment.

![demo v301](/img/pent/v301.png)


### Vul 4: XSS 1 - Search
A reflected cross site scripting vulnerability exists in the search field for the website allowing an attacker to execute arbitrary JavaScript in a client's browser, for instance by giving them a link with crafted parameters.

**Demo:**

1.	Browse to any page with a website search field, such as ```http://websecurity.com/```

2.	Click the magnifying glass up top to open a search field

3.	Place ```; alert(1);``` into the search field and search by hitting enter

4. A popup alert box with 1 will appear

![demo v401](/img/pent/v401.png)


### Vul 5: XSS 2 - Email
A reflected cross site scripting vulnerability is present in the email field of commenting allowing an attacker to execute arbitrary JavaScript in a client's browser, for instance having them submit a form. This piggybacks off the 2.2 SQL Injection 1 - email vulnerability by using the error thrown when bad SQL is given. 

**Demo:**

1.	In a browser other than chrome (Firefox, internet explorer 11 tested) browse to any page you can post comments, for example ```http://websecurity.com/blog/```

2.	Put ```'"<script>alert(1)</script>``` in the email field, fill other required fields with anything

3. A popup alert box with 1 will appear

![demo v501](/img/pent/v501.png)


### Vul 6: Persistent XSS 1 - Messaging
Another XSS vulnerability exists in the subject of message. This attack is more complex than the previous XSS attacks since it requires login to the application first before launching the attack. This can be done by either having a user account or brute forcing passwords. The application doesn&#39;t have brute force protection. After gaining user access, the attacker can send private messages to other users that include XSS attack.

**Demo:**

1.	Login to the websecurity website

2.	Browse to your targets profile http://websecurity.com/members/TARGETUSER/

3.	Hit the Private Message button

4.	Put ```<script>alert(1);</script>``` in the subject field and anything in the comment and hit send.

5. When that user views their message, they will get an alert popup

![demo v601](/img/pent/v601.png)


### Vul 7: Persistent XSS 2 - Long Comments
A second persistent cross site scripting vulnerability can be found from posting comments that are too long and get truncated. This vulnerability found [1] in WordPress and the database is the result of comments being stored in a field 65535 bytes long. If a comment is longer than this, it is truncated when stored. As comments allow certain html tags such as &lt;a&gt;, what may look like a safe and valid to the WordPress filter can become dangerous once truncated.

**Demo:**

1.	Browse to any page you can post comments, for example ```http://websecurity.com/blog/```

2.	Put in the comment field the following:

 ```
 <a title='x onmouseover=alert(1) style=position:absolute;left:0;top:0;width:5000px;height:5000px  AAAAAAAAAAAA[65535A]AAA'></a> 
 ```

3.	Replace the ```[65535A]``` with ```65535 (64k)``` As (or any other character) 
A txt file with ```65535``` As can be found here if required ```http://pastebin.com/9WvS7uLj```

4. A popup alert box with 1 will appear to anyone who mouseover the comment or area below it (due to 5000px by 5000px region)

![demo v701](/img/pent/v701.png)


### Vul 8: Persistent XSS 3 – 4 Byte Characters
Another cross site scripting vulnerability caused by a bug in WordPress due to how it handles 4 byte characters. This vulnerability relies on the submitted data to be truncated this time by exploiting the fact that WordPress doesn’t support 4 byte characters such as &#128169; [POO emoji], and truncates he message after them.

**Demo:**

1.	Browse to any page you can post comments, for example ```http://websecurity.com/blog/```

2.	Put into the comment field the following
```
<a title='x onmouseover=alert(1) style=position:absolute;left:0;top:0;width:5000px;height:5000px 💩'> 
```
3. A popup alert box with 1 will appear to anyone who mouseover the comment or area below it (due to 5000px by 5000px region)

![demo v801](/img/pent/v801.png)


### Vul 9: Persistent XSS 4 - Comment
When an attacker gains access to the website using a user that has Editor authorization level, then the attacker can post a script in the comment field. This XSS attack is persistent and therefore can affect everyone who views the attacked page. A malicious user can also launch this attack. 

**Demo:**

1.	Login into the website

2.	Login into the website with Editor authorized user or admin. I used "Kostya" after brute forcing password.

3.	Post a comment with ```<script>alert(1);</script> ```

4. Any user who views the page with the comment on will have a popup alert box appear 


### Vul 10: Code Execution 1 - Filter 
A severe code execution vulnerability was found in the website. The vulnerability was found in the search field of the members page. Using a proxy, The attacker can change the value of "Filter" to any code to be executed in the server. The impact of this vulnerability is disastrous because the attacker would have control over the application server. This issue requires an immediate solution. 

**Demo:**

1.	Browse to the members page ```http://websecurity.com/members/```

2.	Start and attach proxy 

3.	Put any data into the Search Members text field and search

4.	Intercept request with proxy

5.	Add (or modify) the filter field to be ```cat /var/www/html/wp-config.php``` (or other desired command)

![demo v1001](/img/pent/v1001.png)

![demo v1002](/img/pent/v1002.png)

6. Forward request onto server

7. The code be executed and the contents of the ```wp-config.php``` file will be visible in the response

![demo v1003](/img/pent/v1003.png)


### Vul 11: Code Execution 2 - Profile Images 
Another code execution vulnerability exists in the application but it is more limited to those who have user access or attackers who can gain user access through brute forcing which is not prevented in the application. The vulnerability is in uploading a profile image and the executed code is in the header comment section of the profile picture. This is major exploits and can affect the confidentiality, integrity and availability of data in the application. 

**Demo:**

1.	Create a ```.jpeg``` image file

2.	Use a program that can modify the ```JPEG Exif``` data such as ```jhead```

3.	Modify the comment and put ```echo EXPLOIT;``` (or any other php code you'd like executed)

4.	Login to the websecurity website

5.	Browse to your change profile photo page ```http://websecurity.com/members/USERNAME/profile/change-avatar/ ```

6.	Choose the edited ```.jpeg``` file and hit upload

7. The text EXPLOIT will appear on the page

![demo v1101](/img/pent/v1101.png)



### Vul 12: CSRF
The application is vulnerable to cross-site scripting request forgery (CSRF). An attacker can trick users to perform an undesired action such as posting comments on their behalf in this case. Tricking the user can be done using social engineering. This vulnerability can be seen by observing the behavior of the application as the value of the nonce is modified. This attack can affect integrity and repudiation of the application.

**Demo:**

1.	Browse to any page you can post comments, for example ```http://websecurity.com/blog/```

2.	Start and attach proxy 

3.	Put any data into comments

4.	Intercept request with proxy

5.	Modify ```akismet_comment_nonce``` and/or ```ak_js```

6. With a modified nonce you'd expect comment to be prevented, but it still appears normally

![demo v1201](/img/pent/v1201)


### Vul 13: Man-in-the-middle 
The authentication process in the application is vulnerable to man-in-the-middle attacks since there is no encryption before transmitting data from the browser to the application. SSL ,which is standardly used in most web applications, is not used. Intercepting data can be difficult but when done can have major impact on the application. 

**Demo:**

1.	Browse to the login page ```http://websecurity.com/wp-login.php```

2.	Start and attach proxy

3.	Login with a username and password

4.	Intercept request with proxy

5. You can see that the password in the POST request is plaintext

![demo v1301](/img/pent/v1301.png)


### Vul 14: Visible Index
There are numerous indexes within the website that can be accessed by anyone. Example of such indexes is ```http://websecurity.com/wp-includes/ ```which contains some information about the website. This vulnerability , while not severe, still makes the website exposed to attackers. 

**Demo:**

1.	There are dozens to hundreds of index that can be viewed by a user, browse to any of them. One example is ```http://websecurity.com/wp-content/plugins/buddypress/```

2. You can see the contents of the directory

![demo v1401.png](/img/pent/v1401.png)