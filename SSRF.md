#### 1.Lab: Basic SSRF against the local server

- This lab has an SSRF vulnerability in the stock check feature
- We see that to check the stock a post request is sent in the stockApi section
- now we write `stockApi=http://localhost/admin/delete?username=carlos` instead
- this
#### 2.Lab: Basic SSRF against another back-end system

It is stated in the lab. desc
- Send the `Post /product/stock` request to repeater
- change stockApi value to `http%3A%2F%2F192.168.0.1%3A8080%2Fadmin%2Fdelete?username=carlos`
- send the new POST request to Intruder
- Add a payload to at the end of the IP address `192.168.0.PAYLOAD_GOES_HERE`
- Set the payload to consist of Numbers from 1 to 255
- Wait till you get the 'Congratulations' message

#### 3.Lab: ~~Blind SSRF with out-of-band detection~~ Burp PRO


#### 4.Lab: SSRF with blacklist-based input filter

In this lab it is written in the lab description that the application has two weak SSRF-prevention measures. Prior to this we need to know that these applications might block certain expression in the request ergo blacklist certain words

- change the stockApi section to `stockApi=http://localhost/admin/delete?username=carlos`
- you'll see that this request is blocked by the application
- change the admin part of the stockApi to `%2561%2564%256d%2569%256e`
- and change the localhost part to 127.1
- with these changes the lab will be solved

#### 5.Lab: SSRF with filter bypass via open redirection vulnerability
It is sometimes possible to bypass filter-based defenses by exploiting an open redirection vulnerability. If the redirection is vulnerable we can use to perform an SSRF attack the lab tells us to use this URL : `http://192.168.0.12:8080/admin`.
- try pevious aproaches and fail
- click on the next product link
- check that it has a path section in the url
- change the url to `path=http://192.168.0.12:8080/admin/delete?username=carlos` in the stockApi section
- Voila Lab solved

#### 6.Lab: ~~Blind SSRF with Shellshock exploitation~~ Burp Pro


#### 7.Lab: SSRF with whitelist-based input filter

Some applications may have a whitelist that hold the expressions that are allowed in contrary to blacklist. In these cases we aim to exploit the inconsistencies and things that are overlooked while parsing the URL. We are informed that the we need to use the `http://localhost/admin` URL and it also informs that the vulnerability lies in the stock check feature.

- add a usernam to check if the website still returns a response `https://Wassup@stock.weliketoshop.net`
- add a `#` to express a URL fragment
- you will see that this is blocked by the application
- double URL encode `#` which is `%2523`
- change `Wassup` to `localhost:80` to be seen as someone accessing through the localhost
- now when we put the request together it looks like this:
- `http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos`
- and Voila Lab solved.