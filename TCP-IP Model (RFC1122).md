- TCP/IP (Transmission Control Protocol/Internet Protocol), just Like OSI is also a model that describes how a network should operate. It defines an applicable framework on proper computer network communication unlike its ISO counterpart which is a theoretical model. It smushes the last 3 layers of the OSI model into one application layer. And takes OSI's data link and physical layers and represents them as link layer.
- This model has 4 layers which are:
	- Application Layer which contains protocols such as SMTP, HTTP/s, FTP,SSH,
	- Trasnport Layer which contains UDP and TCP also haivng systems such as TLS to ensure packet security,
	- Internet Layer which houses the famous IP and the ICMP, this layer is responsible with routing and forwarding the packets to the desired destination,
	- Link Layer is te layer that holds the Ethernet and WiFi protocols, it represents the physical aspects of networking and the connection between devices.

RFC (Request For Comment)
- It is a term created by the Internet Engineering Task Force. It is a publication that represents interntet technologies. Such as RFC792 (ICMP) and RFC1034 (DNS).

HTTP AUTH
- IT is the event of a user being authenticated upon requesting to access certain conent.

- Basic Auth
	- Comprises of the usage of a username and a password to authenticate the user's identity. The username and the password are combined with a colon in between them. The resulting string is then encoded in base64 which results with a string similar to this: "dHdpbGlvOmFob3kh".

- Cookie-based Auth
	- When a user opens a website a session cookie is issued to the user. This cookie is then used to authenticate the user's identity.
- JWT authentication
	- This method uses a JSON file for authentication. 
	- It has three parts and they are:
		- Header: holds metadata bout the token such as the algorithm used for encryption.
		- Payload: contains user data.
		- Signature: This signature is obtained through encoding the payload and the hedaer in Base64url then this string is run through the algorithm previously mentioned in the header.
HTTP Methods:
**GET Request**

This is used for getting information from a web server.  

**POST Request**

This is used for submitting data to the web server and potentially creating new records  

**PUT Request**

This is used for submitting data to a web server to update information

**DELETE Request**  

This is used for deleting information/records from a web server.

Common Request Headers:
Host: Tells the server which website is reaquested
User-Agent: containse browser information
Content-Lenght: Tells the server how much data to expect so that no data is missing.
Accept-encoding: Holds the compression method the browser uses
cookie: Data sent to the server to help remember your information

Common response Headers:
Set-Cookie: Information to store 
Cache-control: Duration to hold the request content in the cache
Content-Type: The type of data the server is returning HTML CSS JS images
Content-Encoding: the method that is used to compress the data.
