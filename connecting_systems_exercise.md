Notes on HTTP1.1, the Web, and how servers handle requests over HTTP:
Victor Elgersma
2023-10-24


- Statelessness is a core principle of HTTP
	- The web server doesn't retain any memory of user interaction. Each request from a client to a server contains all the information needed to understand and process that request. 
	- implications of statelessness
		- statelessness simplify server design
		- more reliable, no session-specific errors
		- flexibility - clients can send requests to any server in a distributed system without needing to communicate with a specific server that holds their session state. 
		- *challenges*: when continuity across multiple interactions is required (i.e online shopping) - need technologies built on top of the HTTP protocol like cookies and session persistence.
		- The statelessness of the web plays a crucial role in its scalability, reliability, and simplicity. At the same time, it has motivated a myriad of innovations and workarounds to support applications that require stateful interactions. The balance between stateless foundation and stateful mechanisms defines much of the web's architectural evolution.
	- web primitives:
		- plain text, html, css, xml
		- **HTTP** (viz HTTPS, FTP)
			- set of rules that WWW uses to transfer data, and defines what actions web servers and browsers should take in response to various commands
			- see entire HTTP protocol https://datatracker.ietf.org/doc/html/rfc2616 written by the IEFT ([Internet Engineering Task Force](https://www.ietf.org/))
			- common "verbs" or commands (HTTP protocol):
				- GET
					- retrieve data from a server. request parameters are typically appended to the URL. It is meant to be idempotent
					- Web crawlers like Google's spiders typically use GET requests to index the web. If content or navigation was behind POST requests, it could be inaccessible to these crawlers. 
				- POST
					- **non-idempotent**:
						- meant for submitting data to a server, causing a change in state or side effects on the server. 
					- POST request data is sent in the body of the request, not URL
					- Problems with overusing post
						- POST-based navigation is frustrating to users since using the back button after a POST request can result in warnings about resubmitting form data
				- PUT
					- update a current resource with new data
				- DELETE
					- remove specified resource
				- HEAD 
				- OPTIONS
				- PATCH
				- follows **CRUD** heuristic (Create, Read, Update Delete)
		- **Safe methods** don't request any server change the state of a server (read-only)
			- GET, HEAD, OPTIONS
			- server can still keep a log if it wants to
			- web crawlers rely on calling safe methods
		- **idempotent methods:**
			- making the same request multiple times should have the same effect as making it once. 
			- POST is NOT idenmpotent
	- **Error messages / HTTP status codes**:
		- Every response from the server to the client includes a status code
			- 3xx and 4xx are Error messages
			- note that when I curl google **➜**  **~** `curl -I www.google.com`, the HTTP status code `200 OK` is in the first line (`200` is the status code, and `OK` is the human-readable description of that code. 
		- `1xx` informational
		- `2xx` (successful)
		- **3xx (Redirection)**: Indicates that further action needs to be taken by the user agent to fulfill the request. Common ones include `301 Moved Permanently` and `302 Found` (commonly used for temporary redirects).
		- **4xx (Client Error)**: These are errors caused by the client. The most well-known is `404 Not Found`, indicating that the server couldn't find the requested resource. Another example is `400 Bad Request`, indicating that the request syntax is invalid.
			- 418 - I am a teapot
		- **5xx (Server Error)**: These indicate errors on the server's side. For instance, `500 Internal Server Error` means the server encountered an unexpected condition. `501 Not Implemented` means the server doesn't support the functionality required to fulfill the request
	- Anatomy of an HTTP request:
		- **User action**
			- typically the process starts with a user action, like:
				- entering a URL in the browser's address bar
				- clicking on a hyperlink on a web page
				- submitting a form on a website
				- a `curl` request made from the terminal
		- **DNS Lookup**:
			- Before the browser can send a request to the server, it needs to know the IP address of the server hosting the desired URL
				- The browser first checks its local cache to see if it is already knows the IP address for the domain
				- If not, it queries the DNS server to resolve the domain name to an IP address
		- **Browser initiates connection**:
			- browser initiates a connection to the web server using the TCP protocol. 
				- TCP is a protocol used for error correction
			- Three-way handshake:
				- Browser sends a SYN (synchronize) packet to the server
				- Server responds with a SYN-ACK (synchronize-acknowledge) packet
				- The browser replies with an ACK (acknowledge packet)
		- **Sending HTTP request**:
			- the request consists of :
				- a request like (containing the HTTP verb and the path of the requested resource
				- headers - contains metadata about request, such as the user agent (browser type)
				- body - in requests like POST and PUT, this contains data the client sends to the server (i.e form data)
		- **Server processes the request**
		- **Server sends HTTP response**
			-  HTTP RESPONSES:
				- Can be **cacheable**:
				- A response that can be cached, saving a new request to the server
				- GET and HEAD are common cacheable methods
				- PUT, DELETE, and POST are common non-cacheable methods. 
		- **Browser processes the response**
		- **Closing the connection**
	- Curl
		- While web browsers fetch content and then render it in a user-friendly graphical way (displaying images, formatting text, playing videos), `curl` fetches the raw data without rendering it.
		- Does not follow redirects by default
- **The Web vs HTTP:** 
	- if WWW is a vast library, HTTP is the communication protocol between librarians and patrons. It's the method by which you request a book and the librarian understands and fulfills that request.
- World Wide Web
	- designed by Tim Berner's Lee as a way to meet the increasing demand for information-sharing among physicists in universities and institutes worldwide
	- anatomy:
		- URL (Uniform Resource Locator)
		- HTML (HyperTest Markup Language)
		- Web Browsers and Web Servers
	- SGML (Standard Generalized Markup Language)
		- 1980s
		- huge influence on the web's foundational markup language
		- meta-language that allows users to define their own markup languages, especially for documents
		- HTML was an application of SGML as a specific markup language for creating web documents. 
- **scaling the client -server model**:
	- getting new hardware (vertical scaling)
	- horizontal scaling 
		- new challenges:
			- need **Load balancer** to route the request
				- do we do the decryption at the Load balancer or at the individual servers?
					- if decryption is done at the LB, we create a DMZ (demilitarised zone)
				- *reliability vs scalability tradeoff*
					- when we introduce a load balancer, we introduce a SPOF (Single point of failure)
					- solution: add another load balancer?
						- new problem: two LBs need to share state, SSL private key now needs to be installed in two different places
				- **session stickiness**:
					- users requests always redirected to the same backend server
					- **Load balancing algorithms**:
						- round robin
						- least connections
						- fastest response
						- fixed weighting
						- hash-based load balancing
					- can be implemented at the network layer (4) or the application layer (7)
						- layer 4 load balancing:
							- forwarding decisions are based on the IP address and port numbers
							- at this level, the load balancer does not inspect the content of the message - it doesn't know if its HTTP or FPT etc
							- sticky sessions using source IP persistence
					- Application layer (7) load balancing:
						- Content-based routing:
						- Layer 7 load balancing can make decisions based on the content of the request
							- e.g route requests for images to one server and requests for database queries to another
					- can inspect HTTP headers & payload.
					- SSL decryption can be handled at the load balancer
						- creates new security vulnerabilities
					- advanced "sticky session" capabilities such as cookie-based session persistence.
	- OSI Model (Open Systems Interconnection)
		- a *conceptual framework* that standardizes the functions of a telecommunication or computing system into seven distinct layers, and is useful for understanding *network concepts*
		- L1: Physical Layer
		- L2: Data Link Layer
		- L3: Network Layer
		- L4: Transport Layer
			- ensures data trasnfer is reliable, ordered, and error checked.
			- ports segments (TCP), datagrams (UDP), flow control, error checking, TCP, UDP
		- L5: Session Layer
		- L6: Presentation Layer
		- L7: Application Layer
			- **function**: interface between the network and the application software running on a computer. 
			- **Key Concepts**: HTTP, FTP, SMTP, POP3, DNS.



# Sources


HTTPv1.1 protocol
https://datatracker.ietf.org/doc/html/rfc2616

mozilla development docs
https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods

curl documentation:
https://curl.se/








