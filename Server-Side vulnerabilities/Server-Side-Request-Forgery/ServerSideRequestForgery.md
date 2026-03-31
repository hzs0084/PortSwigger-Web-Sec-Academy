# SSRF

Cause the server side application to make requestes to an unintended location

Typically, it's to an internal only services within the organization's infrastructure. In other cases, to an external systems to leak sensitive data like authroization creds. 

Thes issues occur because of how trust relationships are handled and requests orginating from the local machine are handled differently than ordinary requests

1. [Basic SSRF against the local server](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/ssrf-apprentice/ssrf/lab-basic-ssrf-against-localhost)

Browse to /admin and observe that you can't directly access the admin page.

Visit a product, click "Check stock", intercept the request in Burp Suite, and send it to Burp Repeater.

Change the URL in the stockApi parameter to http://localhost/admin. This should display the administration interface.

Read the HTML to identify the URL to delete the target user, which is:
`http://localhost/admin/delete?username=carlos`

Submit this URL in the stockApi parameter, to deliver the SSRF attack.

Used [URL Decpde/Encoder](https://meyerweb.com/eric/tools/dencoder/) to encode the payload

