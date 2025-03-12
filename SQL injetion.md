PortSwigger cheat sheet for SQL Iinjection
https://portswigger.net/web-security/sql-injection/cheat-sheet

SQLi refers to being able to run sql in input fields or in the url to manipulate the database the website is using.

There are several types of SQLi

Error-Based SQLi
- in this type, the website returns the result of yout querry, be it successful or an error. therefore we can detect that there is a SQLi vulnerability
- ' OR 1=1 --
Union-Based SQLi
' Union select username, password from users where username ='admin' --

Blind SQLi
Boolean Blind SQLi
send a payload to check the response of the application
' And 1=1 --

Time-Based Blind SQLi
use delays to understand the behaviour of the application
' OR IF(1=1, SLEEP(5), 0) --


Abi ben Protswiggerdan 10 lab çözdüm de not almadım