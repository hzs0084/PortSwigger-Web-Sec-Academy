# GraphQL API Vulnerabilities

*Befores testing GraphQL API, you need to find its endpoint. GraphQL APIs use the same endpoint for all requests*

## Universal Queries

Sending `query{__typename}` to any GraphQL endpoint, it will include the string `{"data": {"__typename": "query"}}`

## Some Common endpoint names


- `/graphql`
- `/api`
- `/api/graphql`
- `/graphql/api`
- `/graphql/graphql`

Appending `/v1` to the path also helps

*GraphQL services respond to any non-graphQL request with a "query not present" or similar error.*

Best practice for prod GraphQL endpoints is to only accept POST requests that have a content-type of `application/json` as it helps to protect against CSRF vulnerabilities. 

Other endpoints may accept alternative methods such as GET requests or POST requetst that use a content-type of `x-ww-form-urlencoded`


