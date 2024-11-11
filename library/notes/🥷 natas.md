

- robots.txt -> contains access details of sites file system for crawlers like google. [ref](https://chatgpt.com/share/3b02ea8b-91c9-40a4-bc44-ee1975be1580)

- relative paths: image se file system ki access lelo via urls, `http.../files/img.png -> http.../files/`
- headers and cookies modify kar sakte ho. natas [4](https://hackmethod.com/overthewire-natas-5/?v=06fa567b72d7),[5](https://hackmethod.com/overthewire-natas-5/?v=06fa567b72d7) 
- php files are not viewable bcuz only html is in inspect, butkahi se woh miljaye toh passwords ya other sensitive info ka relative paths mil jata hai.
- [natas7](https://hackmethod.com/overthewire-natas-7/?v=06fa567b72d7) -> [local file inclusion vulnerability](https://wiki.owasp.org/index.php/Testing_for_Local_File_Inclusion) -> `http://natas7.natas.labs.overthewire.org/index.php/index.php?page=home` was the given link and `hint: password for webuser natas8 is in /etc/natas_webpass/natas8`. seems like i can pass path of a file to view it(page=filepath). so i justs put in the path of the passwd, page=/etc/natas_webpass/natas8 in url!
- natas9 -> php sourcecode available; `passthru` is php function to run cli commands, seeing that somewhere opens up the possibility for command injection. here the page takes user input and runs it using passthru,
	![[Screenshot 2024-05-27 at 1.11.40 AM.png]]
	now maybe i just pass in `natas9.natas.labs.overthewire.org/?needle=foo;cat /etc/natas_webpass/natas10 &submit=Search`  (the foo; part is to fillin whatever was actually intended and run the next command)
	this will make the cli of server run cat /etc/natas_webpass/natas10 showing me the token for nats10!!
- natas10: this was  the src code. `passthru("grep -i $key dictionary.txt");` 
  exploited the fact that we can grep multiple files together. [link](https://hackmethod.com/overthewire-natas-10/?v=06fa567b72d7)