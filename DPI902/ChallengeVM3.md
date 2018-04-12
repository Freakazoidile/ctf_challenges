# DPI 902 - Challenge VM 3
**Submitted By:** Thomas Zuk

**Challenge Machine IP**: 172.16.11.69


## Solution
This challenge was a web-only challenge so no port or service scanning was performed. 

Visiting the website `http://172.16.11.69` yields a "under construction" site page.  Looking at source of the page there is no useful information, just a few pictures that make up the page. The page `172.16.11.69/robots.txt` contains 2 interesting entries "flag.php" and "key2".

The flag.php page displays an empty array() and a message that says keys unlock the flag, visiting the page again the array is popular with the key:pair values `key1:LOCKED, key2:LOCKED, key3:UNLOCKED`

The page "key2" redirects to the home page.  I decided to try visiting "key1" and "key3". Key1 is a directory which has 2 files "key1/magic_word.php" and the other is an image, which is used on the magic_word page. The image is of the character Nedry from Jurassic Park  with the message "you didn't say the magic word".


Looking at the request and response in Burp suite there is a cookie aptly named "magicword", I changed the value of the cookie to "please" (the magic word) which was met with a new response "Thanks but I only speak EBCDIC".

EBCDIC is a type of encoding, convert "please" to EBCDIC results with "p%e/se", placing this value into the magicword cookie gives `KEY1: WhereWeAreGoingWeDontNeedRoads`

Looking more closely at /Key2/ in Burp suite reveals there is a seemingly blank page that redirects back to the home page. Upon further insepction there is actually a large amount of blank space on the Key2 page (more than 1 screens worth), and later on a form with a single input "command" and a HTML comment "command getflag added".

Forging a POST request to the key2 page with contents `command=getflag` results in Key2 `AllTheThings`

Visiting the flag.php page again we see that Key1 and Key2 remain locked and Key3 is unlocked. I tried replacing Key1 and Key2 cookies through burpsuite with the values "UNLOCKED" but this did not give the Flag. 
I replaced the respective key1 and key2 values with the values of the key I obtained so that I created a request to flag.php as follows:


![alt text]( https://raw.githubusercontent.com/Freakazoidile/ctf_challenges/master/DPI902/screenshot%204.png "Challenge VM3 - Flag")

As can be seen, this gives the flag!

`Flag: MayTheCybersBeWithYou`







