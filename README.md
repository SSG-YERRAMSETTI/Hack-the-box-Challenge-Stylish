# Hack-the-box-Challenge-Stylish
Stylish is a hackthe box web challenge 
We Noticed Something Weird (Bug in the App)
the website uses a thing called Content Security Policy (CSP). It tells the browser which resources (fonts, scripts, images) are allowed fonts to be loaded from any website or server
It had a dangerous setting:
Content-Security-Policy: font-src 'self' *;

This is very dangerous — because now we can use CSS tricks to spy on the admin.

What’s the Plan?

•	The admin looks at your card submission.
•	There's a secret code (approvalToken) in their view.
•	We can’t see it, but…
•	We can make the admin’s browser "leak" each letter of that code to us by loading fonts from our server.
That's the main trick here.

So we used a creative hack using fonts and CSS to steal a hidden code (called approvalToken) from the admin’s page.

There’s a hidden field in the admin’s browser (only viewable when the admin visits your page), which contains a secret token.
So when the admin sees our submission:
•	If the code has A, we see a request on our server for .../leak/A
•	If it has B, we see .../leak/B
•	And so on...
We do this for A-Z, a-z, and 0-9 — 62 total letters.
Each letter leaks one character from the hidden code.

We Used a Python Script (solve.py) to Do It All
This script does 2 things:
1.	Sends the card with the special CSS to the website.
2.	Runs a small server that listens for leaked characters (like A, b, 3, etc.)
As the admin views the page:
•	Their browser loads fonts.
•	Our server sees which characters got loaded.
•	We slowly rebuild the secret code, one letter at a time.

![image](https://github.com/user-attachments/assets/c8c1d92d-c73a-40e3-9a53-50d59904a76c)


Now We Have the Secret Code (approvalToken)

This code is required to unlock or approve the card submission.

Only the admin is supposed to have it.
But now we have it — and we didn’t even need to see the page. We just let the browser leak it to us.

Why localhost?
•	The approval system only works on the admin’s local machine (127.0.0.1 = localhost).
•	But by tricking the admin's browser into making the request, we bypass the protection.


We Use the Token to Trick the Admin Into Approving Our Card
With the secret code in hand, we use another trick:
We ask the admin’s browser to visit a special internal URL with the token.
Like:
http://127.0.0.1:1337/approve/1/the_token
This makes the admin automatically approve our card



Important Trick: The admin’s browser is running locally — we can’t access 127.0.0.1, but the admin can. So we force them to access it via CSS!
1.	sqlmap reads the req_entries.txt file.
2.	It finds the pagination parameter (which is user-controlled).
3.	It sends payloads like:
{"pagination":"10' OR '1'='1"}
4.	It watches how the server responds:
o	If the response changes in a suspicious way → it's probably injectable.
5.	Once confirmed, it can:
o	List tables
o	Dump data (like the flag!)
o	Even run SQL commands directly
In this challenge, pagination was passed unsanitized into a SQL query, so sqlmap could inject easily!

