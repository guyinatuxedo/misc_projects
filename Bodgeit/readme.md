#This is a writeup from the vulnerable web app Bodegit. I did not make the app.

#1.) 
You know how your programming teacher said to always comment your code? That can be bad. While on the main page press “Ctrl” + “u” to look at the source code and look for anything of interest. Comments can be fairly interesting regardless of what your programming teacher made them look like. Remember an html comment starts with 
```
<!--
```
And ends with
```
-->
```
After three seconds with “Ctrl” + “f” we do find something of interest.
```
“<!-- td align="center" width="16%"><a href="admin.jsp">Admin</a></td→”
```

You see this looks like a piece of html code that got commented out. It appears to be a hyperlink to an admin page named “admin.jsp”. Let’s try to access it! Wow that is a lot of data to have fun with. 

Yourip + ”/bodgeit/admin.jsp”

#2.)
Now that we have all of this data we will do something fun with it. We will update someone else’s basket. To start this go to the “Your Basket”. Now you will need to fire up Tamper Data.  Next you will need to press the “Update Basket” button, and then you will need to tamper with the data. There will be a field named ‘id’. Now I wonder what would happen if you changed that to “1” or “2”?

You are essentially updating someone else’s basket by switching the basket id.

#3.)
You see that Search tab? It seems lonely. We should pay it some company. By clicking on it it reveals that it is a typical search bar. By typing in “Doodahs” it reveals that it is indeed a search bar. Let’s try to inject some html code into it! First try the following.

```
<u>Gizmo</u>
```

And it’s output is 

```
You searched for: Gizmo
```

Now let’s try to insert this code

```
<script>alert("XSS")</script>
```

Wow look at that beautiful message box.

#4.)
Now continuing with our theme of XSS we find ourselves at the “Contact Us” page. Let’s try the same attack again using “<script>alert("XSS")</script>”.

And our output is
```
alert(XSS)
```

It appears that our double quotes and our script tags have been removed, thus stopping that attack. This is done by a server side script that is looking for certain characters and just replacing them I wonder what would happen if we used single quotes and upper case script tags instead.

```
<SCRIPT>alert(‘XSS’)</SCRIPT>
```

And now we get to see that wonderful pop up.

#5.)
Now let’s try to get the store to owe us money. First let’s try to use tamper data to alter the price of the items were adding to the cart. The value we would be looking for is Price.

So much for that. But there is another approach. Instead of altering the price we could try altering the quantity. The value we would be looking for is Quantity.

Success. The store now owes us money.

#6.)
The next challenge is to try to change the password via a get request. First let’s see how to password is currently being changed. To do this, we can add the Live HTTP  Headers addon.
https://addons.mozilla.org/en-US/firefox/addon/live-http-headers/

Now you just have to start it, then change the password. When you look at the output you see that the password is being changed with a POST request. Now let’s change that. We can do this with an add on called FireBug, which allows us to easily edit html code.
https://addons.mozilla.org/En-us/firefox/addon/firebug/

Now we can just right click on that form, and inspect it with Firebug. We can change it to a GET request from a POST by changing this

```
<form method=”GET”>
```

to

```
<form method=”POST”>
```

#7.)
Let’s access another person’s basket. Stores store their customer’s session in cookies for convenience purposes. We will be editing that cookie. We will use CookieEditor to edit the “b_id” value to 1. Then you will just need to update the basket and you would have successfully updated your basket.

#8.)
Keep in mind this exercise is bugged and will provide the correct results when not done properly.

helpful to us. But it can just be downright confusing to end users. That’s why most web devs disable it. However the devs could still use it (alot). So the solution to this is they disable it by default, yet have a variable set in place to enable error messages. They do this by adding a Get request through the URL. There are some examples below.

```
phobetor.xyz/fail.php?debug=start
phobetor.xyz/fail.php?debug=true
phobetor.xyz/fail.php?state=debug
phobetor.xyz/fail.php?state=test
```

Here the debug value is just named debug. Now we need to create an error. How could we break something? Let’s try updating the basket, and swapping out the item number with a letter. Now it will have to find what X is. Also you need to set the debug variable by inserting the variable in the url.

```
192.168.154.356/bodgeit/basket/jsp?debug=true
```

Now we can change the values using tamper data and boom we have an error.

#9.)
Next we are going to try to do an sql injection. We are going to attack the login page. A lot of password functions of websites function off of a sql query. So let’s just try to feed it a simple single quotation. 

Take note of the upper left hand corner. We got an error message. That mean that it is vulnerable to SQL injection. So the sql query probably looks like something like this

```
Select * from users where username=’$username’ and password=’$password’; 
```

We have several different usernames, however we don’t have any passwords. Good thing is we can insert an or statement into it, thus overriding the need for a password. Let;s say that we are trying to login to the account test@bodgeitstore.com. We would put this

```
test@thebodgeitstore.com’ or ‘1’=’1
```

Thus converting the query to 

```
select * from users where username=’test@thebodgeitstore.com ‘ or ‘1=’1’ and password=’’;
```

Since the first condition is met, we are authenticated successfully. Now just do it with the other user accounts (user1@thebodgeitstore.com and admin@thebodgeitstore.com).

#10.)

Next we are going to use XSS to change the main admins password. Since we already have access to the main admin, it will be a bit easier. When we login to the admin account, the “contact us” page turns into “Comments”. That means any XSS attacks that we place in the “Contact Us” page will be viewed by the admin. We know that the password can be changed by a GET request. This makes things much simpler. If we take a look at the change password screen we can use Firebug and LiveHTTP Headers to tell exactly what is changing the password. In this case it is 

```
http://192.168.206.136/bodgeit/password.jsp?password1=IceNineKills&password2&Ice Nine Kills
```

Looking at the code, we can tell that the html tags being used for the password change is the <img>. So if we stick it in we get

```
src='http://192.168.206.136/bodgeit/password.jsp?password1=penis5&password2=penis5'>
```

The closing tags are located later on in the code,s o we don’t need to worry about it. Now we just have to post that as a comment, and when the admin views the page he will have his password changed.

#11.)

There is one page we haven’t visited yet, the advanced search page “127.0.0.1:8080/bodgeit/advanced.jsp”. Let’s check out the source code and see if anything interesting comes up.

  There is a javascript script and function being running whenever something is submitted through the form.

  There is a hidden input box titled q hidden in form tags

  The Javascript script loads an external encryption file, and does some encryption work itself.
  
Now let’s try to analyze what happens the script by giving it a random input.

```
<SCRIPT>
    loadfile('./js/encryption.js');
    
    var key = "9892797b-c441-41";
    
    function validateForm(form){
        var query = document.getElementById('query');
        var q = document.getElementById('q');
        var val = encryptForm(key, form);
        if(val){
            q.value = val;
            query.submit();
        }   
        return false;
    }
    
    function encryptForm(key, form){
        var params = form_to_params(form).replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;').replace(/'/g, '&#39');
        if(params.length > 0)
            return Aes.Ctr.encrypt(params, key, 128);
        return false;
    }
    
    
    
</SCRIPT>
```

First, the function loads a script which sets the necessary variables and functions for AES encryption.

The key variable is used to store the AES encryption key.

The query variable reads the second form on the page

The q variable reads the hidden element

The function below that parses the input, and then replaces certain characters that we need to hack it


:) 
Next let’s try to impersonate a user! First you are going to need a user account, whether it be the one it gives you or one you made on the spot. Then sign in, and make sure that it remembers your credentials. Now comes the fun part. Fire up Cookie Editor and filter the cookies using the ip address 









