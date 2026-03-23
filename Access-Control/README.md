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

# Labs

1. [Unprotected Admin Functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)

Navigate to /robots.txt and then find the dashboard /administrator-panel

Delete the user carlos

2. 