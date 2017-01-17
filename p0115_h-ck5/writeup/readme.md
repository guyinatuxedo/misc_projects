#Objective: 
You are given a vm. On the vm there is a web server (apache). Log on as guest, get the ip address then on your host machine type in the ip address into a web browser. When you get there you see your objective.

Welcome to the polls. This is a place where you can vote for your favorite meme. Your objective is to hack and completely own the polls. You will make the list in this order, from most votes to least: Forever Alone, Harambe, Simpsons, Doge, Confession Bear, Rule 34, Tumblr, 4chan, Bertstrips, Dick Butt, pepe, dump, wallpaper dump, Philosoraptor, Good Guy Greg, Bad Luck Brian, Ancient Aliens, Success Kid, Dat Boi, Penguin. After you are finished, send a screenshot to Ryan. After that he will confirm the results.

#Solution:
    Let’s see what happens when we vote for Forever Alone. To do this, we will need some sort of proxy. Pick one whether it is Burp, Tamper Data, etc. When we use one, we see that it is posting a variable known as “elect” with a value of “Forever+Alone” (the plus is there because of the space). With this it is safe to assume the script that it is posting to just takes whatever candidate is in this variable and adds a vote. To verify this, let’s change “Forever+Alone” to “Pepe”. If our theory is right it should just add a vote to Pepe. And it does.

    Now to get the objective done, there are lots of different ways we can do this. We could attempt to attack the database and update the values. We could create a script that scrapes the website, determines what categories need how many votes, than throw hundreds of thousands of votes until we have powned the script. Spoiler Alert we are going with the later. 

    The first thing we need to know is the backend php script that we will be posting he elect variable to. Chances are our proxy will tell us. For Tamper Data it tells us in the header.
    
    ```
    “http://172.16.213.132/vote.php”
    ```
    
    Of course your ip address will depend on your vmware network settings, however the script name “/vote.php” should remain the same. There is another way we can find this. We can just look at the http source code for the voting page. And it tells us here
    
    ```
    “<table border="2px solid black" width="100%"><tr><th>Meme</th></tr><form action="vote.php" method="post">”
    ```
