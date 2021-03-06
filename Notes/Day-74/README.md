## Cue Section
1. Differences between Authentication and Security.
2. What's CSRF and how to prevent it? (2 approaches).
4. What's XSS and how to prevent it? (2 approaches).
5. How to avoid XSS if we want to output the raw content?

---

# Website Security
> To prevent unexpected actions by visitors, potential malicious actions like exposing data or granting unwanted access to certain actions on our website.

## [CSRF](https://portswigger.net/web-security/csrf) Attacks
> It stands for Cross Site Requesst Forgery, simply refers to attacks that forces an end user to execute unwanted actions on a web application in which they're currently authenticated. With a little help of social engineering (such as sending a link via email or chat).

> Depending on the nature of the action, the attacker may trick the users to perform state changing requests like transferring funds, changing their email address, and so forth.

## How CSRF works?
> A user clicked on a link shared by the attacker which lead to the fake webpage which contains a from to perform data input or other actions, with the help of browser automatically attaching the cookies, after submitting the form, it will redirect to the authentic website and it also recognized the user's authenticated and so as the action that has been changed by the attackers.

## To Prevent CSRF
### [SameSite](https://simonwillison.net/2021/Aug/3/samesite/)
> The approach is to set a `SameSite='xxx'` attribute in the cookies where it stores in the client-side browser, when the a submitting form is redirecting to other page, the cookies won't be automatically attached in the request header which prevented the default browser behavior.
#### Potential [Disadvantages](https://www.ibm.com/docs/en/cdfsp/7.6.1.x?topic=checklist-vulnerability-cookie-without-samesite-attribute#:~:text=Vulnerability%20Details,script%20inclusion%2C%20and%20timing%20attacks.)
1. Links redirected from other website to the authentic page will be not authenticated and requires login everytime.
2. Potentially, there might be browser with older version not supporting SameSite cookies attribute.

### CSRF Token
> The best way to prevent CSRF is to generate a CSRF token for every form that are being sent as a POST request, the server will check whether the form is authenticated as well with matching the CSRF Token behind the scenes.

> It make sure the user came from a form on your own site, and not a form hosted somewhere else.
---

## Implementing CSRF Token in Express.js
1. Install the third-party package
```console
npm install csurf
```
2. Require the package
```js
const csrf = require('csurf');
```
3. Use `csrf` as the middleware
```js
app.use(csrf()); // the default configuration can do the trick
```
4. Generating CSRF Token for every Form
```js
router.get('/transaction', function(req, res) {
  if(res.locals.isAuth) {
    const csrfToken = req.csrfToken(); // generating Token in every request cycle
    res.render('transaction', { csrfToken: csrfToken });
  }
});
```
5. Add the Token into the Form
```js
// transaction.ejs
<form action="/transaction" method="POST">
  <input type="hidden" value="<%= csrfToken %>" name="_csrf" />
  //...
  <button class="btn">Send</button>
</form>
```
> The `name="_csrf"` attribute is neccessary for the csrf middleware to works, it will automatically look into the input where name is `"_csrf"`.

---

## [XSS](https://portswigger.net/web-security/cross-site-scripting#:~:text=How%20does%20XSS%20work%3F,their%20interaction%20with%20the%20application.) Attacks
> It stands for Cross Site Scripting, when a website is able to upload data and share to other users, you can upload some JavaScript for say to perform certain action. Since the website is going to output the uploaded data, it also execute the script as well.

## XSS [Examples](https://infinitelogins.com/2020/10/13/using-cross-site-scripting-xss-to-steal-cookies/)
> XSS can be used to steal the Session Cookie where the attackers are able to get access to your account and get full control of it.

## To Prevent XSS
### Escaping from XSS
> It's the primary means to prevent XSS attacks by telling the browser to treated the data as plain text and should not be interpreted in any other way.

### Sanitize Content
> If you don't want to escape the content, sanitizing is another way to prevent XSS attacks, by encoding the content in a way that HTML or CSS can render as a plain text instead of markup and script. ([Not recommended](https://benhoyt.com/writings/dont-sanitize-do-escape/))

---
