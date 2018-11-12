# CS 4173 Lab 4
#### Cross-Site Request Forgery Attack Lab

## Prerequisites
Versions of software used listed here:
- Environment: SEEDUbuntu 16.04

## Task 1: Observing HTTP Request

In this task, we use the provided tool to generate two binary files which result in the same hash when using the MD5 hashing algorithm. By observing the output, we can see that the prefix is appended to the output binary as a 64 length block. There are 3 prefix files included: `63longprefix.txt`, `64longprefix.txt`, and `65longprefix.txt`. These were used to test the outputs when the prefix is a multiple of 64, less than a multiple of 64, or greater than a multiple of 64. 

## Task 2: CSRF Attack Using GET Request

In this task we will demonstrate a GET request attack to show how a GET request attack can be launched if the victim is directed to a malicious site. We can create a webpage in the directory for the csrflabattacker.com domain which has a hidden image with the `src` set to the URL we wish to attack and the GET request will be triggered automatically. The image tag used is the following: 

```html
<img src="http://www.csrflabelgg.com/action/friends/add?friend=42" alt="" style="position:absolute; left:-999999999999999999999999999999999999999px" width="1" height="1">
```

## Task 3: CSRF Attack Using POST Request

