## Regex

Regex (Regular Expressions) are essentially searches on steroids. You can search for verry ambiguous things, and can specify your search parameters with much more detail then just a string, and a lot of other stuff. Let's get into it.

#### 0.) Same String

The most basic Regex is just searching using a string. For instance the string "guy" is an acceptable paramter. It will then just search for the characters "g", "u". and "y" in that specific order and case.

String:
```
decisionsguymadeforme
```

Regex:
```
guy
```

Result:
```
guy
```

#### 1.) Various Combinations

With Regex, you can specify that certain characters in a string can be various different characters. For instance you can specify if the g in "guy" could be upper case or lower case, so "guy" and "Guy" would both be acceptable. In addition to that, you could specify a different character completely, so you can have different words qualify for a search such as "auy", "buy", and "cuy".

String:
```
guy Guy guY gUY GUY 
```

Regex:
```
[gG]uy 
```

Result:
```
guy Guy
```

String:
```
guy Guy guY gUY GUY auy buy cuy duy
```

Regex:
```
[a-c]uy
```

Result:
```
auy buy cuy
```


#### 2.) Digits

You can use regex to search for indivual digits, by using the "\d" option. 

String:
```
I @m 1337 h@c50r
```

regex:
```
\d
```

Result:
```
1 3 3 7 5 0
```

Now if you want to go more specific, let's say only the digits 5 through 7

String:
```
I @m 1337 h@c50r
```

Regex:
```
[5-7]
```

Result:
```
7 5
```

Now let's say that you want to find only ones, threes, and zeroes

String:
```
I @m 1337 h@c50r
```

Regex:
```
[130]
```

Result:
```
1 3 3 0
```

#### 3.) Quantifies

White regex, you can specify how many times you will want a string. This c

