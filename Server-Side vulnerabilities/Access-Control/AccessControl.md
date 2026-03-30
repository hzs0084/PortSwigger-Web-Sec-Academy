# Access Control Vulns and Priv Esc

Access control is how the application decides who or what is authorized to perfom actions or get access to resources that are available on the application. 

It depends on:
- Authentication: Confirms to see the user is who they say they are
- Session Management: Which subsequent HTTP requests are being made by the same user
- Access Control: Is the user allowed to carry out the action they are trying to perform?

## Access Control Security Models

1. Programmatic Access Control
2. Discretionary Access Control
3. Mandatory Access Control
4. Role-based Acces Control

### Vertical Access Control

Different types of users have access to different application functions

Example:

Alice -> Regular User (Can't perform functinons like Bob)

Bob -> Admin User (Can modify or delete users)

It's broken when Alice  can elevate her permissions to become an Admin user. 

### Horizontal Access Control

These control access to resources to specific users.

Example:

Alice (Regular User) -> Can modify and make transactions only on her account

Bob (Regular User) -> Can modify and make transactions only on his account

It's broken when Alice can modify Bob's transactions. 

### Context-depedent Access Control

Prevent's users to perform actions in the wrong order. 

Example, a retail website prevents users from modifying the contents of their shopping cart after they have made the payment. 

### Parameter-based access control methods

Some applications determine the user's access rights or role at login, then store that information in an environment that is user controleld. It could be a cookie, hidden field, or a preset query string paramter. 

`https://insecure-website.com/login/home.jsp?admin=true`\
`https://insecure-website.com/login/home.jsp?role=1`

Modifying requests can help users elevate their privileges.

# Labs - Access Control

1. [Unprotected Admin Functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)

Navigate to /robots.txt and then find the dashboard /administrator-panel

Delete the user carlos

2. [Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)

Reading the source code, tells you the admin enpoint - `/admin-wuwugk`

3. [User role controlled by request parameter](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-role-controlled-by-request-parameter)

Change the role to admin from false to true

4. [User ID controlled by request parameter, with unpredictable user IDs](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)

Intercepting the requests and then changing the parameters of the postId, leaks the admin and carlos's user id as the post was made by them 

taking those user id's, i changed the requests during the login process and got his API key as the solution

5. [User ID controlled by request parameter with password disclosure](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure)

Changing the id fiels, to carlos, and to administrator reveals the password value hiddein in the form 

use that to login as admin and then go to admin panel to delete carlos


# Labs - File Upload Vulnerabilities

# Labs - OS Command Injection

# Labs - SQL Injection

