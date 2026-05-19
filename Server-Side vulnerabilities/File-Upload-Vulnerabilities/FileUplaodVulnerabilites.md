## What are File Upload Vulnerabilities?

When a web server gives a user the capability to upload a file to the server without validating things like the file's name, type, contents, or size. The upload functionality can be misused to upload dangerous files like server-side script that enable remote code execution

Then the following action with a HTTP request for the file, which forces the server to execute the file. 

## How do these vulns arise?

A dev may think that they have validated enough checks on the back end but with any blacklisting, it is easy to bypass it and miss out on obscure file types that are dangerous. 

The checks that are put in place can also be manipulated by an attacker using Burp Proxy or Repeater. 

Inconsistent validation across different hosts on the network can also be exploited. 

## Exploiting unrestricted file uploads 

When a server let's you upload server-side scripts like PHP, Java, or Python files, it is also configured to execute them as code. 

Getting a webshell is then trivial with one-line php code

One example for reading files can be

```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```

Once uploaded a request for this file will return the target file's contents in the response

----
A more versatile one-liner would be

```php
<?php echo system($_GET['command']); ?>
```

Using a HTTP request

```http
GET /example/exploit.php?command=id HTTP/1.1
```

## Lab

### Remote Code Execution via Web Shell Upload

- I browsed around the website to see where the upload functionality lives
- I couldn't find anything on the site so I logged in with the username and password `wiener:peter` 
- I noticed the avatar upload feature
- I used the php one liner and then uploaded it 
- Since I used the command one, I changed the HTTP request to type in my commands
- I learned that for space i need to use the `%20` encoding 
- also the parameter command `?command=` 
- `?` is essential for the query
- `command` is how the script is going to get the variable and execute it, it could be anything
- i went to the dir `/home/carlos/secret` and tried to cat the output but i thing i noticed that anything that I typed in through the get parameter, i would get two responses
- so the cat output was concatenated and i didn't realize it until i looked at the solution

#### Community Solution

- this actually uses the path to the target file liner and then you call on to it directly
- i don't think i mentioned this above but the sitemap helped me learn where the file exists as in the location that i need to ping to get the php to execute. 

Adding a test here to ensure the changes take place in the repo




