---
title: Switch to HTTPS Now, For Free
author: admin
type: post
date: 2013-09-30T11:12:25+00:00
url: /archives/14474
categories:
 - 系统架构
tags:
 - https
 - ssl

---
From now on, you should see a delightful lock ![](https://konklone.com/assets/images/blog/https/ssl-0-lock.png) next to [https://konklone.com][1] in your browser’s URL bar, because I’ve switched this site to use HTTPS. I paid **$0** for the trouble.Why you should bother doing the same:



 * SSL’s not perfect, but we need to [make surveillance as expensive as possible][2]
 * For privacy not to be suspicious, [privacy should be on by default][3]
 * And hey, bonus: [more complete referrer information][4] in Google Analytics for people visiting from sites already using HTTPS (like [Hacker News][5]).

This post shows how to do your part in building a surveillance-resistant Internet by switching your site to HTTPS. Though it takes a bunch of steps, each one is very simple, and you should be able to finish this in **under an hour**.


A quick overview: to use HTTPS on the web today, you need to obtain a certificate file that’s signed by a company that browsers trust. Once you have it, you tell your web server where it is, where your associated private key is, and open up port 443 for business. You don’t necessarily have to be a professional software developer to do this, but you do need to be **okay with the command line**, and comfortable configuring**a web server you control**.

Most certificates cost money, but at Micah Lee’s [suggestion][6], I used [StartSSL][7]. They’re who the [EFF][8] uses, and **their basic certificates for individuals are free**. (They’ll ask you to pay for a higher level certificate if your site is commercial in nature.) The catch is that their website is difficult to use at first — especially if you’re new to the concepts and terminology behind SSL certificates (like me). Fortunately, it’s not actually that hard; it’s just a lot of small steps.

Below, we’ll go step by step through signing up with StartSSL and creating your certificate. We’ll also cover installing it via nginx, but you can use the certificate with whatever web server you want.

## Register with StartSSL {#register-with-startssl}

To get started, [visit their signup page][9] and enter your information.

![](https://konklone.com/assets/images/blog/https/ssl-1-signup.png)

They’ll email you a verification code. They tell you to **not close the tab** or navigate away from it, so just keep it open until you get the code, and can paste it in.

![](https://konklone.com/assets/images/blog/https/ssl-2-signup-verify.png)

You’ll need to wait for certification, but it should only take a few minutes. Once you’re approved, they’ll email you a special link and a verification code to type in.

That’ll bring you to a screen to generate a private key. They’re generating you this private key inside your browser, using the [“keygen” tag][10]. However, **this isn’t the key you use to make your SSL certificate**. They’re using it to create a separate “authentication certificate” that you will use to log in to StartSSL’s control panel going forward. You’ll make a separate certificate for your website later.

![](https://konklone.com/assets/images/blog/https/ssl-3-auth-key-generate.png)

Finally, they’ll ask you to “Install” the certificate:

![](https://konklone.com/assets/images/blog/https/ssl-4-auth-cert.png)

Which installs your authentication certificate directly into your browser.

![](https://konklone.com/assets/images/blog/https/ssl-5-auth-cert-installed.png)

If you’re in Chrome, you should see this at the top of your browser window:

![](https://konklone.com/assets/images/blog/https/ssl-6-auth-cert-chrome-install.png)

Again, this is just the certificate that identifies you by your email address and lets you log in to StartSSL going forward.

Now, we need to get StartSSL to believe we own the domain name we want to generate a new certificate for. From the control panel, click the “Validations Wizard” tab, and select “Domain Name Validation” from the dropdown.

![](https://konklone.com/assets/images/blog/https/ssl-7-begin-domain.png)

Enter your domain name.

![](https://konklone.com/assets/images/blog/https/ssl-8-choose-domain.png)

Next, you’ll select an email address that StartSSL will use to verify you own the domain name.

As you can see, StartSSL will believe you own the domain if you control webmaster@, postmaster@, or hostmaster@ with the domain name, **OR** if you own the email address listed as part of the domain’s registrant information (in my case, that’s currently `konklone@gmail.com`). Choose an email address where you can receive mail.

![](https://konklone.com/assets/images/blog/https/ssl-9-choose-domain-email.png)

They’ll email you a validation code, which you can enter into the field to validate the domain.

![](https://konklone.com/assets/images/blog/https/ssl-10-domain-validate.png)

![](https://konklone.com/assets/images/blog/https/ssl-11-domain-done.png)

## Generating the certificate {#generating-the-certificate}

Now that StartSSL knows who you are, and knows you own a domain, you can generate your certificate using a private key.

While StartSSL **can** generate a private key for you — and their FAQ assures you they use only the [highest quality random numbers][11] and [don’t hold onto the key][12] afterwards — it’s also easy to create your own.

This guide will cover creating your own via the command line. If you choose to let StartSSL’s wizard do it, you can pick back up with this guide a couple steps down, where you choose the domain the certificate should apply to.

To create a new 2048-bit RSA key, open up your terminal and run:

`openssl genrsa -aes256 -out my-private-encrypted.key 2048`

You’ll be asked to choose a pass phrase. [Pick a good one][13], and **remember it**. This will generate an _encrypted_ private key. If you ever need to transfer your key, via the network or anything else, use the encrypted version.

The next step is to decrypt it so that you can generate a “certificate signing request” with it. To decrypt your private key:

`openssl rsa -in my-private-encrypted.key -out my-private-decrypted.key`

Now, generate a certificate signing request:

`openssl req -new -key my-private-decrypted.key -out mydomain.com.csr`

Go back to StartSSL’s control panel and click the “Certificates Wizard” tab, and select “Web Server SSL/TLS Certificate” from the dropdown.

![](https://konklone.com/assets/images/blog/https/ssl-12-cert-begin.png)

Since we generated our own private key, you can hit “**Skip**” here.

![](https://konklone.com/assets/images/blog/https/ssl-13-cert-key-skip.png)

Then, paste in the contents of the .csr file we generated earlier.

![](https://konklone.com/assets/images/blog/https/ssl-14-csr-enter.png)

If all goes well, it should say it received your certificate signing request.

![](https://konklone.com/assets/images/blog/https/ssl-15-csr-received.png)

Now, choose the domain you validated earlier which you plan to use with the certificate.

![](https://konklone.com/assets/images/blog/https/ssl-16-choose-domain.png)

It requires you to add a subdomain. I added “www” for mine.

![](https://konklone.com/assets/images/blog/https/ssl-17-choose-subdomain.png)

It will ask you to confirm. If it looks right, hit “Continue”.

![](https://konklone.com/assets/images/blog/https/ssl-18-cert-ready.png)

**Note**: It’s possible you’ll get hit with a “Additional Check Required!” step here, where you wait for approval by email. It didn’t happen to me the first time, but did the second time, and my approval arrived in ~30 minutes. Upon approval, you’ll need to visit the “Tool Box” tab and visit “Retrieve Certificate” to get your cert.

That should do it — your certificate will appear in a text field for you to copy and paste into a file. Call it whatever you want, but the rest of the guide will refer to it as`mydomain.com.crt`.

## Installing the certificate in nginx {#installing-the-certificate-in-nginx}

First, make sure **port 443 is open** on your web server. Many web hosts automatically keep this port open for you. If you’re using Amazon AWS, you’ll need to make sure your instance’s Security Group has port 443 open.

Next, we’re going to create the “certificate chain” that your web server will use. It contains your certificate, and StartSSL’s intermediary certificate. (Including StartSSL’s root cert is not necessary, because browsers ship with it already.) Download the intermediate certificate from StartSSL:

`wget http://www.startssl.com/certs/sub.class1.server.ca.pem`

Then concatenate your certificate with theirs:

`cat mydomain.com.crt sub.class1.server.ca.pem > unified.crt`

Finally, tell your web server about your unified certificate, and your decrypted private key. I use nginx — below is the bare minimum nginx configuration you need. It redirects all HTTP requests to HTTPS requests using a 301 permanent redirect, and points the server to the certificate and key.

[shell]

server {

listen 80;

server_name konklone.com;

return 301 https://$host$request_uri;

}

server {

listen 443 ssl;

server_name konklone.com;


ssl_certificate /path/to/unified.crt;

ssl_certificate_key /path/to/my-private-decrypted.key;

}


# for a more complete, secure config:

# https://gist.github.com/konklone/6532544

[/shell]


[view raw](https://gist.github.com/konklone/6416795/raw/11c48f580a64b6ad068f45c61ccab8b2630ed8a0/konklone.conf) [konklone.conf](https://gist.github.com/konklone/6416795#file-konklone-conf) hosted with ❤ by [GitHub](https://github.com/)

You can also check out [a more complete HTTPS nginx configuration](https://gist.github.com/konklone/6532544) that turns on [SPDY](http://www.chromium.org/spdy/spdy-whitepaper), [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security), SSL [session resumption](https://code.google.com/p/sslyze/wiki/SessionResumption), and enables [Perfect Forward Secrecy](https://www.eff.org/deeplinks/2013/08/pushing-perfect-forward-secrecy-important-web-privacy-protection).Qualys’ SSL Labs offers an excellent [SSL testing tool](https://www.ssllabs.com/ssltest/analyze.html) you can use to see how you’re doing.


Now, ensure your nginx configuration is okay (this also verifies that the key and certificate are in working order):

[shell]sudo nginx -t[/shell]

Then restart nginx:

[shell]sudo service nginx restart[/shell]

Cross your fingers and try it out in your browser! If all goes well, the ![](https://konklone.com/assets/images/blog/https/ssl-0-lock.png) will be yours.

## Mixed Content Warnings {#mixed-content-warnings}

If your site is running on HTTPS, it’s important to make sure all linked resources — images, stylesheets, JavaScript, etc. — are HTTPS too. If they’re not, users’ browsers will complain. Newer versions of Firefox will [outright block insecure content][14] on a secure page.

Fortunately, pretty much every major service with embed code has an HTTPS version, and most (including Google Analytics and Typekit) handle it automatically. For others, you’ll need to figure it out on a case by case basis.

## Back up your keys and certificates {#back-up-your-keys-and-certificates}

Don’t forget to **back up your SSL certificate, and its encrypted private key**. I put them in a private git repository, and included a brief text file describing every other file, and the process or command that created it.

You should also **back up your authentication certificate** that you use to log in to StartSSL. StartSSL’s FAQ [has instructions][15] — it’s a .p12 file containing a cert + key that you export from your browser.

**More Resources**

[Troy Hunt posted](http://www.troyhunt.com/2013/09/the-complete-guide-to-loading-free-ssl.html) his own guide to HTTPS with StartSSL for Windows and Azure users.

[Glenn Fleishman’s 2009 article](http://arstechnica.com/security/2009/12/how-to-get-set-with-a-secure-sertificate-for-free/) covers setting up HTTPS with Mac OS X and Apache.

[Simon Westphahl’s 2012 blog post](http://www.westphahl.net/blog/2012/01/03/setting-up-https-with-nginx-and-startssl/) describes setting up HTTPS using the command line and nginx.

More discussion on this over at

[Hacker News][16] and [Reddit][17]. from:

 [1]: https://konklone.com/
 [2]: http://www.theguardian.com/world/2013/sep/05/nsa-how-to-remain-secure-surveillance
 [3]: http://www.tbray.org/ongoing/When/201x/2012/12/02/HTTPS
 [4]: http://stackoverflow.com/a/1361720/16075
 [5]: https://news.ycombinator.com/
 [6]: https://twitter.com/micahflee/status/368163493049933824
 [7]: https://www.startssl.com/
 [8]: https://www.eff.org/
 [9]: https://www.startssl.com/?app=11&action=regform
 [10]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen
 [11]: https://www.startssl.com/?app=25#43
 [12]: https://www.startssl.com/?app=25#44
 [13]: http://xkcd.com/936/
 [14]: https://blog.mozilla.org/tanvi/2013/04/10/mixed-content-blocking-enabled-in-firefox-23/
 [15]: https://www.startssl.com/?app=25#4
 [16]: https://news.ycombinator.com/item?id=6340825
 [17]: http://www.reddit.com/r/webdev/comments/1luu5w/switch_to_https_now_for_free_a_stepbystep_guide/