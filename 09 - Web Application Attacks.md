# WebApp assessment methodology

## WebApp #enumeration
ID the components that make up a webapp before attempting to blindly exploit it (***ENUMERATION***) 
- The stack in use could include info such as programming language and frameworks, web server software, db software, and server OS (**wappalyzer**, **whatweb**).
1. Inspecting URLs -- file extensions can reveal the programming language of the application (**.php**).  This is becoming less common.
2. Inspecting page content -- #devTools display page resources and conent
3. Viewing response headers -- Can use a #proxy ( #BurpSuite pro/community or OWASP ZAP) or DevTools Netowrk tool to view HTTP requests/responses.  #headers that start with #X- are non-standard HTTPheaders which reveal addtl info about the tech stack used by the app.
4. Inspecting #sitemaps -- The two most common are **robots.txt** and **sitemap.xml** 
	1. **curl https://www.google.com/robots.txt**
5. Locating admin consoles -- common examples include the *manager* application for Tomcat (hosted at /manager/html) or *phpmyadmin* for MySQL (hosted at /phpmyadmin).

## WebApp assessment tools
Assessment tools can aid in disocring and exploiting webapp vulns.  Many are pre-installed in Kali.
1. #DIRB -- web content scanner that uses a wordlist to find directories and pages.  DirBuster is a java application similar to DIRB that offers multi-threading and a GUI-based interface.
2. #BurpSuite -- GUI-based and a powerful proxy tool. ***Burp Pro is prohibited during the exam, but not community***
3. #Nikto -- Open Source web server scanner.  High performance, but not designed for stealth (embeds info about itself in the *User-Agent* header.)
4. #Autorecon

## Exploiting web-based vulnerabilities
1. Exploiting admin consoles -- #defaultCreds  are the first and simplest exploit to check.  Be careful not to overdo it and alert the blue team.  **Burpsuite Intruder** can help with this by automating basic username and password combinations.
2. XSS -- root cause of #XSS is *data sanitization*.... or lack thereof.  When unsanitized input is displayed on a web page, this creates XSS.  Most common characters to check for XSS are < > ' " { } ;. If input is passed onto the output in DevTools, a vuln exists. Three variants : 
	1. #Stored (Persistent) -- exploit payload stored in the DB or cached by a server.  Attacks all users of the site ; comment sections  product reviews.
	2. #Reflected -- payload is in a link and only attacks the person viewing the link.  Often occur in search fields and results, places were user input is included in error messages
	3. #DOM-based --Take place solely within the page's Document Object Model. In short, a browser parses a page's HTML content and generates an internal DOM representation.  JS can programmatically interact with this DOM, and so DOM-based occurs when a page's DOM is modified with user-controlled values.
	Can use XSS in combo with an iframe to redirect legimitate traffic to an attacker platform.  iframe is used to embed another file within the current HTML document. Can also steal #cookies and #session info. Two particularly interesting *optional cookie flags*
	1. #secureFlag : instructs the browser to only send the cookie over encrypted connections (prevents cleartext cookies captured over the network)
	2. #HttpOnly flag : instructs the browser to *deny* JavaScript access to the cookie (allws us to steal the cookie with XSS payload if **not** set) by crafting a payload, and then reading the cookie wen a user connects
	![[Pasted image 20221025105923.png]]
	![[Pasted image 20221025110333.png]]
3. #directoryTraversal Vulns (Path Traversal) -- Can expose sensitive WebServer information, but do not exeute code on the application server.  Best to use files that are world-readable (**``/etc/passwd``** on Linux or **``c:\boot.ini``** / **``c:\windows\system32\drivers\etc\hosts``** on Windows)
4. #fileInclusion Vulns -- Often in unison with Directory Traversal vulns. Must be able to execute code ***AND*** zrite our shell payload somewhere.  #LFI --> the included file is loaded from the same webserver.  #RFI --> the included file is loaded from an external source.
	1. Can be exploited with log-file poisoning (LFI)
	2. For RFI vulns, PHP apps must be configured with *allow_url_include* set to "On"
	3. #HTTPServer in Python3 : **python3 -m http.server 8888 bind 127.0.0.1**
		1. HTTP server in php : **php -S 0.0.0.0:8888**
		2. #PHP wrappers -- use to exploit directory traversal and LFI vulns
			1. *data wrapper* - embed inline data as part of the URL with plaintext or *base64* encoded data (alternative payload).  Called using "data:" followed by the type of data (**data:text/plain,**) 
			2. After the comma, you can use php-encoded commands.  As an example : 
![[Pasted image 20221025142521.png]]
5. #SQLi -- caused by unsanitized user queries bieng passed to the DB for execution.  SQLi can be identified by using a single quote ('), a SQL string delimiter. To protect against SQLi, you'll want to put into place *parameterized queries* *Note*: for Blackbox testing, you'll need to input a quote into every field that we suspect may pass a parameter to the DB. SQLi can lead to : 
		1. Authentication bypass - ***NOTE*** - you may have to use the "LIMIT" clause to get a result that doesn't throw an error. 
		2. Use the "order by" clause to enumerate the number of columns, starting with 1 and continuing until you get an error.  (***This can be automated using Burp's repeater***)
		3. SQLi to Code Injection -- 




