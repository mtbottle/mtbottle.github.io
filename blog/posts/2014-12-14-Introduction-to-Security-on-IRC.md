---
title: Introduction to Security on IRC
---
I started using IRC a couple of months ago as an alternative to chatting with friends to google talk. The other reason is that I should probably start getting familiar with it as there are lots of development resources that I am current not tapping due to being ignorant of how to use IRC. I have been fairly phobic of public forums after a couple stints in my youth (long story short: I no longer go on random RPG chatrooms, anime forums or ask people who wants to chat about Diablo what it is). And since I've heard of many hackers being on IRC, I was understandably scared about identity theft and getting hacked. So this post is a foray into me making my IRC experience more secure.

<!--more-->
<h1>Running IRC on VPS</h1>
I had dabbled with IRC during my anime downloading days (before bittorrent became popular), but had never really learned to use it. Then I dabbled on and off for certain FOSS projects I wanted to be part of. My original setup included a desktop client (mIRC or xchat) and connecting locally from my laptop or PC. However, the there are certain limitations to having a locally running client, which is solved when running the IRC client on a VPS.

Since I have a free VPS with computing resources, I decided that I want to host my client connection on that remote host. One of the advantages of hosting remotely is having a machine on all the time (since I travel around so much, I now only own a laptop) which allows me "historical" (read: lurking) access to the chat content when I'm away. Therefore, I started using <a href="http://irssi.org">irssi</a> which is a command line IRC client. To ensure that I have it up all the time, I use <a href="http://linux.die.net/man/1/screen">screen</a> and detach and reattach the irssi session when I need to log off/on my VPS. This is a fairly typical setup for anyone who have IRC running on a VPS.
<h2>Other Ways to Mask Your IP</h2>
Another possible advantage of using a VPS is that hackers do not get easy access to your computer's ip address, which gives you a level of indirection. However, the hacker does have access to your VPS' ip, so if you have anything incriminating on it, then you are screwed anyways. One way to obfuscate your ip (if you are careful) is to use <a href="https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO/irssi">Tor with irssi</a>.

Another popular way to mask your ip is by using an IRC bouncer like znc. A bouncer will essentially offset your IRC connections to a remote host, kind of like what I'm already doing running a detachable irssi on a remote host. However, one advantage to using a bouncer is that you can pick up your IRC messages on any device. Znc enables a web interface which is useful if you want to be able to IRC on your phone (if you don't want to install a terminal on your phone... and there's good reasons as there aren't any good terminal apps for android). In my case, since this solution only gives one level of indirection which I already have with my VPS, I have no need to use a bouncer.

The last way you can hide your ip is by setting up <em>cloaking.</em> Cloaking basically tells the IRC server to associate your hostname with whatever you want to identify it with when someone types in <code>/whois</code> on your name. This is set up on a server by server basis, so you will need to contact the admins for the particular network you're connected to set this up.

The most secure way to obfuscate your ip is to set up Tor with irssi on my VPS. I have not set this up because I don't feel the need to at this point. If I ever do, it will be the subject to another blog post... [TBD, but laziness]
<h1>Setting up [freenode] SSL for Irssi</h1>
Most connections on irc are plaintext, which means that it is susceptible to being snooped on. This is why if the IRC server allows for SSL connections, it should be used. So this section will discuss how to set up SSL connections for setting up on freenode (might be extendable to other chat servers as well). Of course, I didn't figure out how to do this myself (well sort of); I just followed the tutorials <a href="https://freenode.net/certfp/makecert.shtml">here</a>, <a href="https://freenode.net/certfp">here</a>, and <a href="https://freenode.net/certfp/certfp-irssi.shtml">here</a>.
<h2>Create the Self-signed SSL Certificate</h2>
Rather than paying for a trusted certificate from a proper certificate authority, I opted to set up my SSL connection with a self-signed certificate (which costs me nothing). Since I'm using this only for encryption purposes (I'm&nbsp; logged in as myself on my VPS and myself on freenode, so I don't need freenode to verify me as myself), a self-signed certificate should be sufficient.

So to create a self-signed certificate:
<pre>openssl req -newkey rsa:2048 -days 730 -x509 -keyout [NAME_YOUR_KEY].key -out [NAME_YOUR_CERT].cert -nodes</pre>
&nbsp;This creates a 2048-bit certificate which will expire after 730 days. I also added <code>-nodes</code> to disable passphrase as I found that caused problems with irssi. Then I concatenated the key and cert files into a pem:
<pre>cat [NAME_YOUR_CERT].cert [NAME_YOUR_KEY].key &gt; [NAME_YOUR_PEM].pem</pre>
&nbsp;The pem file will be used by irssi to allow for encryption. To make sure that the file is used by irssi, move the file into your <code>~/.irssi</code> directory. Now, assuming that you have an account on NickServ, you will need to get your .pem fingerprint so that freenode can identify you. To get your fingerprint type in:
<pre>openssl x509 -sha1 -noout -fingerprint -in mynick.pem | sed -e 's/^.*=//;s/://g;y/ABCDEF/abcdef/' &gt; fingerprint</pre>
Then <code>cat fingerprint</code> will give you the fingerprint hash that freenode can associate your NickServ account with your SSL certificate:
<pre>/msg NickServ cert add [FINGERPRINT]</pre>
<h2>Setting up Irssi to sign in with SSL</h2>
Now that we have our SSL certificates and have connected them with our account on freenode, we can set up irssi to automatically sign in to freenode using SSL. I opted for strict SSL validation (so that on my server's end that I can validate that freenode is actually freenode), which involves downloading <a href="http://www.instantssl.com/ssl-certificate-support/cert_installation/UTN-USERFirst-Hardware.crt">UTN-USERFirst-Hardware certificate</a>. I then copied the certificate <code>~/.irssi/UTN-USERFirst-Hardware.pem</code>.

From the tutorial, you can add a freenode connection with this command in irssi:
<pre>/network add -whois 1 -msgs 4 -kicks 1 -modes 4 freenode
/server add -auto -ssl -ssl_cert ~/.irssi/[NAME_YOUR_PEM].pem -ssl_verify -ssl_cafile ~/.irssi/UTN-USERFirst-Hardware.pem -network freenode chat.freenode.net 6697</pre>
This will add a block in your <code>~/.irssi/config</code> file under "servers" that looks like:
<pre>{
    address = "chat.freenode.net";
    chatnet = "freenode";
    port = 6697;
    use_ssl = "yes";
    ssl_cert = "~/.irssi/[NAME_YOUR_PEM].pem";
    ssl_verify = "yes";
    ssl_cafile = "~/.irssi/UTN-USERFirst-Hardware.pem";
    autoconnect = "yes";
}</pre>
So you should be able to change any of the ssl configurations in your irssi config file.

Although we have set up SSL encryption on our end, unless everyone else you are chatting with are also using SSL there are still ways for hackers to get your messages by listening to an unsecure connection from someone in the channel. Therefore although setting up SSL should be done, it is not a completely foolproof.
<h1>Disabling and/or Avoiding DCC</h1>
There have also been recommendations to disable <a href="https://en.wikipedia.org/wiki/Direct_Client-to-Client">DCC</a> (direct client-to-client) as it is potentially insecure as these messages bypass the IRC server altogether (and if you're on a server that masks your ip, then this can potentially reveal it). DCC mostly used for file transfer, so if you know you aren't going to be transferring or receiving files, then you should disable DCC. Irssi doesn't have a setting to disable DCC, but have <a href="http://www.irssi.org/documentation/settings">configurations</a> which allows you to configure how you want DCC to work. The best way to ensure safety is to not send files over IRC and to not autoaccept download requests. Irssi configurations sets dcc_autoaccept to be false by default, so it shouldn't be an issue.
<h1>Conclusion</h1>
All the things I mentioned in this article are precautions you can take to ensure safety when using IRC. Of course, even if you take all the precautions, including setting up Tor, but stupidly give out personal information (like address or <a href="http://www.bash.org/?244321">password</a>), it will render all those precautions useless.


