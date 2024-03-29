John the Ripper


#How does it work?


John the Ripper (JTR) is a password cracking tool. It is available for OSX, Linux, and Windows. It was originally designed to crack Unix passwords, but now it can crack things such as passwords for zip files. In order to use JTR you need the password hash. A hash is a string of text that has been ran through a hashing algorithm. A hashing algorithm has to meet the two following requirements theoretically.

```
Password hashes cannot be reversed
For every input, there is exactly one output
```

If the algorithm meets those two requirements, then it should be a hashing algorithm. Since the hash cannot be reversed, that means if the authentication server is hacked and all of the hashes are leaked then you shouldn’t be able to reverse the hash back to the original password, thus the password remains safe. Secondly since there is only one output for every input, you have to input the right password to get the right hash which will allow for correct authentication.


However it has been found with some hashing algorithms like MD5, that there are multiple inputs that produce the same hash. These are known as collisions. In addition to that, although the hashes cannot be directly reversed it is possible to create a database of hashes and compare hashes to that. For instance, let’s say we hash the letter “a” and add it to a database. If we find a hash somewhere and look it up in the database, and we find a match for “a” then we know that the value of the hash is “a”. Because of this, new hashing algorithms have to be made in order to provide stronger algorithms


JTR operates by attempting to brute force a hash by generating hashes to match it. If run by it’s default setting it will try three different attacks. 


Single-Crack Mode: This uses the username of the account and tries different variations of that to guess the password. Any Passwords successfully guessed are tried against all other accounts.


Wordlist: In this mode JTR uses the default wordlist it comes with (about 4,000 entries) to try to guess the password.


Incremental: In this mode it will try all possible string combinations. It will start at “a”, then “aa”, then “ab” and so on and so forth until it finds the right hash.


Keep in mind that this is just the default settings. You can edit and change this to fit your specific needs.


#Installation


This will cover how to install it on Ubuntu. To install it on other platforms, refer to google. You can install either the basic edition, or the community enhanced edition which comes with special features. To install JTR basic edition run the following command.

```
sudo apt-get install john
```

Now to install the community enhanced edition, it is a bit more involved. You can either download the .tar.gz (or other system compatible file, however the setup might be different) from either the main website (listed below) or from my github.


http://www.openwall.com/john/


First navigate to the directory that the .tar.gz file is stored in using the “cd” command followed by the file path. After that, you can extract the file with the following command.

```
tar -xvzf john-1.8.0-jumbo-1.tar.gz
```

After that a new folder will be created called “john-1.8.0-jumbo-1”. You will need to enter that directory with the following command.

```
cd john-1.8.0-jumbo-1
```

Now this package comes with a wealth of documentation from operation to flags. You can just refer to the Install instructions that reside at “john-1.8.0-jumbo-1/doc/INSTALL” with nano (a basic text editor) using the following command (provided you ran the previous). Attached in the github you will find a copy of the “INSTALL” file, just incase for whatever reason it disappeared.

```
sudo nano doc/INSTALL
```

Usage Example 1


To start off with the Ubuntu example, first we need to get the password hash. With the modern versions of Ubuntu, passwords are stored in a file located at “/etc/shadow”. To view this file type in the following command.

```
sudo cat /etc/shadow
```

From here you will see a list of every user on your system, including daemons. You can create a new user to do this with (and preferably give it a week password). But for this example we will be working with a user named “test” with the following information from the shadow file.

```
test:$6$jlNTBb4A$Ejy1eguZIC1POEaSCJUNzLOGK0DxLXcwI1L1LDbZBWD3cI5pl3ql3KSHWH8vpA0fMexScdEHPCkB/9Ay8xYh5.:17110:1:30:7:::
```

Now in reality it isn’t colored like that. However I color coded it so you can better understand the individual components of this.


The name of the user.
The current hash (encrypted password) of the user
Days since Jan 1, 1970 that the password was last changed
This is the minimum amount of days between password changes for this user.
This is the maximum amount of days allowed between password changes for this user.
This is the amount of days before the next mandatory password change that a warning is issued


Now if you notice, there are still three more collons. Those collons are for two fields, which don’t apply to this account. The first is the number of days after their password expires that their account is disabled, thus ensuring that if an account doesn’t change their password it isn’t breached. The second keeps track of the date in which the account was disabled (it does that by keeping track of the amount of days since January, 1 1970). If you have more questions about this refer to the following link.


http://www.cyberciti.biz/faq/understanding-etcshadow-file/


Now that we have covered the parts of a “shadow” hash, it is time to break it. First we need to copy and paste the hash into a text file for JTR. To do this, you want to copy everything and give JTR as much info to work with as possible, as JTR will be able to utilize a lot of this information to expedite the process. To make the text file you can just copy and paste the colored hash above into a new file (for demo purposes we will name it passwords).


Now that we have the password in a file, we can try to run JTR. To do so on this file run the following command.

```
sudo john passwords
```

Now after a second it comes back with this success message.

```
Loaded 1 password hash (crypt, generic crypt(3) [?/64])
No password hashes left to crack (see FAQ)
```

Now to see all of the cracked hashes from a file, you can use the following command. 

```
john passwords --show
```

And this is the output. 

```
?:Test305555

1 password hash cracked, 0 left
```

Now let’s say you have an idea of what the password is, or you have a better word list for your purposes besides what comes default with JTR. You can instruct JTR to use a specific word list versus the default. To do that you will just need to add an additional flag. There is an example command below, the file that has the hash is called ”wordlist_passwords” and the file that has the new wordlist is named “list”.

```
john --wordlist=list wordlist_passwords
```

And we can see from checking the file it has found and decoded the hash.

```
john --show wordlist_passwords
```
```
test1:TaiFood:17110:1:30:7:::


1 password hash cracked, 0 left
```

Now keep in mind, this write up only covers a small part of what JTR can do. 

