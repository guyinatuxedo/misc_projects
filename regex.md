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
decisions**guy**madeforme
```
