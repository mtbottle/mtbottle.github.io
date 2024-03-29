---
title: Setting up my own Email Server
author: michtran
---
I wasn't going to write a post about setting up my own email server (as there's millions of tutorials online and it's not <em>that</em> difficult to do. However, I ran into some issues in the setup that I would probably want to note to myself if I could go back in time. So here are the notes on setting up my own private mail server using Postfix, Dovecot, on an Ubuntu 12.04 server. This post will mostly be links to the sources that I used and notes on particular problems that I spent a lot of time on.

<!--more-->

Most of my set up was based on : <a href="https://help.ubuntu.com/community/MailServer">https://help.ubuntu.com/community/MailServer</a>, so you can look there for the practicals.
<h1>Setting up the MTA and SASL</h1>
The main mail handling component is called an MTA (Mail Transfer Agent). This process handles all the incoming and outgoing mail from the server. It sets up the SMTP server in which you can send and receive mail (ie. the basic functionality of a mail server).

In the case of my setup, I used Postfix as my MTA and installed it with :
<pre>sudo apt-get install postfix</pre>
I won't go into details about the installation (as you can read it in the link above). For security, I used SASL (detailed instructions can be found in the tutorial as well), and I installed it with :
<pre>sudo apt-get install libsasl2-2 sasl2-bin libsasl2-modules</pre>
By default, port 25 is where all the unsecure connections for the mail server is made. This is used mainly for receiving mail that should be routed to the current server's domain (in this case michtran.ca). For outgoing mail (or routing mail where it's not set to relay to the current domain), you <em>can</em> set it to route unsecurely through port 25, but that would likely put you on a <a href="http://mxtoolbox.com/blacklists.aspx">server blacklist for spam</a>. In fact, in the short time I've had my email server, I've gotten relay requests like :
<pre>Jan 22 22:50:17 mt postfix/smtpd[9232]: connect from 36-224-128-188.dynamic-ip.hinet.net[36.224.128.188]
Jan 22 22:50:17 mt postfix/smtpd[9232]: NOQUEUE: reject: RCPT from 36-224-128-188.dynamic-ip.hinet.net[36.224.128.188]: 554 5.7.1 &lt;gk49fawn@yahoo.com.tw&gt;: Relay access denied; from=&lt;z2007tw@yahoo.com.tw&gt; to=&lt;gk49fawn@yahoo.com.tw&gt; proto=SMTP helo=
Jan 22 22:50:17 mt postfix/smtpd[9232]: lost connection after RCPT from 36-224-128-188.dynamic-ip.hinet.net[36.224.128.188]
Jan 22 22:50:17 mt postfix/smtpd[9232]: disconnect from 36-224-128-188.dynamic-ip.hinet.net[36.224.128.188]</pre>
Therefore, it is important that you disable relay from unsecure connections in your MTA configs. The documentation for ensuring that relay and unauthorized connections are rejected is in <a href="http://www.postfix.org/SMTPD_ACCESS_README.html">this</a> documentation.

Outgoing mail is generally set to route through secure port 465 unless you make your server an open relay (and hence a spam server, which as mentioned above is a bad idea).
<h1>Dovecot: MDA and IMAP/Pop3 Interfaces</h1>
To be able to access and receive mail from a MUA (mail user agent, like thunderbird), we need to set up an imap and pop3 interface on top of the mailboxes we have already set up. This interface is called an MDA (mail delivery agent). The one that I use is <a href="http://dovecot.org">dovecot</a>. Most of the instructions to set up dovecot can be found in <a href="https://help.ubuntu.com/community/Dovecot">this</a> tutorial.

I had particular trouble setting up the outgoing session for dovecot (especially since I wanted to ensure that I could connect to an outgoing session with SASL authentication. There are many misleading information when I was looking for a solution to the problem, but it was mostly resolved with <a href="%20http://wiki.dovecot.org/HowTo/PostfixAndDovecotSASL">these</a> changes to the configurations.
<h1>Passing SPF and setting up MX on domain</h1>
Lastly, to make sure that the mail is routed correctly, I had to set up the <a href="https://en.wikipedia.org/wiki/MX_record">MX</a> (mail exchange) configurations on my domain. MX basically allows for you to host your email on a different machine than the one for your domain. I had to first specify the ip of the machine to route from the subdomain mail.michtran.ca, then set mail.michtran.ca as the MX of the root domain (michtran.ca).

I also noticed that most of my outgoing mail was heading to the spam inbox of the recipient. Upon inspection of the header of the email, I noticed that it was failing SPF:
<pre>Received-SPF: fail (google.com: domain of email@michtran.ca does not designate [omitted] as permitted sender) client-ip=[omitted];</pre>
SPF, which stands for <a href="https://en.wikipedia.org/wiki/Sender_Policy_Framework">Sender Policy Framework</a>, is essentially a domain check for the recipient to ensure that the incoming mail is not being relayed and is verified to be associated with the domain in the address. This is updated in the domain registry which allows it to be looked up when the client receives the email.

To set up the spf record, you need to add a line that looks like this in your domain TXT/SPF configuration:
<pre>"v=spf1 mx a ~all"</pre>
The line above just essentially specify that any mail coming from your domain or your MX servers are legitimate ips for forwarding mail. However, in my case, setting the domains I ran into issues with SPF where it will pass some of the times, and fail most other times. In the cases that failed, I would usually get something that looked like an IPv6 address. This was when I realized that my machine was configured with 16 IPv6 addresses that was not specified in the domain settings. Therefore, to fix this issue I had to disable all the IPv6 forwarding (or add them to my MX record, which would've been more of a hassle).
<h1>Conclusion</h1>
This is the end of this entry for configuring a mail server. The actual postfix server took me about half a day to configure with spamassassin, clamv and amavis antivirus. Dovecot took me more time to understand (maybe a week to properly understand how it connects with postfix) and hence was a bit more work to set up properly with outgoing SMTP functionality. Lastly, the SPF record kept failing with unfamiliar IPs, which took me a while to connect with my IPv6 settings on my machine (so in total about another week). Is it worth the hassle of setting up my own mail server? I would have to say that I learned a lot about how mail servers work (MTA, MUA and MDA layers), but I find myself slightly apprehensive every time I tail the mail.log file. So far, I haven't been hacked, so that's a good sign... for now...

&nbsp;


