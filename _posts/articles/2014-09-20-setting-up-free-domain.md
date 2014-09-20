---
layout: post
title: "How to set up your own domain for free"
modified:
categories: articles
excerpt:
tags: [free, hosted email, domain, DNS, DDNS]
image:
  feature:
date: 2014-09-20T14:02:14+02:00
comments: true
share: true
---

In this post I try to describe the way you can set up domain, subdomain and hosted email service w/o going into deep details, e.g. which registrar, dns provider etc. Try to picture four parties yourself: your domain provider, your DNS provider, your server, and your hosted email service. Well the last one can be broken into two if your domain is cheap like mine. The game now may **begin**!!

## Domain

To have your own free domain is an easy task. Either it is free to register with your internet provider or you can choose a free one like me (dot.tk), which is infamous for abuse and spams :D But it contains beautifully my initials. In the first case your internet provider is the domain provider.

I now assume you already have the domain *mydomain.ltd* as your own.

## DNS Provider

DNS provider is important to set up your subdomain, redirects, emails etc. and maybe dynamic DNS (ddns) if you want to host your own server like me, whose IP changes daily.

With my domain provider I can set up those DNSs except an TXT record for gunmail's domain key with the host name *krs._domainkey*, either because of the period or the underscore. I don't really know and care, now. 

Take note that domain providers can be DNS providers, but they don't always give you the option to set up the DNS records you want so much. So look around.

### Name Server

A domain provider like **1&1** may not support DNS management, but they definitely let you change the name servers. It means you give a DNS provider the permission to manage DNS records of your domain.

First step is to set the name servers (have to > 1) at your domain provider to tell the requests to look up at... something likes *ns.dnsprovider.ltd*. The 2nd step is to create an account at *dnsprovider.ltd* and to register *mydomain.ltd*. After that you can add the DNS records. Beware it can take up to 48 hours to propagate. In my case with *system-ns* nothing happened :(

### ddns

With my cheap Popo I can host my own server. Now I want that *mydomain.ltd* points to its IP. If you don't have a server with a dynamic IP you can skip this section. ddns doesn't need the domain to be added to your dns provider.

Because my domain provider doesn't have an API to update my IP, I found [system-ns](http://system-ns.com/) as the community of my popo's OS [recommended](https://wiki.archlinux.org/index.php/Dynamic_DNS). But after two days waiting for the propagation of my email's records I gave up and ended up with [cloudns](http://cloudns.net). Look at the answer to the question [**Do you offer dynamic DNS and how to use it?**](https://www.cloudns.net/faq/#q12) you can find out how to retrieve an URL which you can invoke an POST method with. Do that as a cronjob hourly or less.

### Sub/domain

After specification your domain has to point to an A/AAAA record, i.e. an IP. With my ddns and cloudns it wasn't a problem. Subdomain however can point to a CNAME record, e.g. another sub/domain, with which the DNS server can look up further for an IP. Able to create such subdomains is important for my Popo to serve different subdomains with its virtual hosts (vhosts). You can read more about it in my other [post](https://blog.hoangkim.tk/ghost/editor/3/). 

CNAME record isn't a redirection, so your url can stay like *host.mydomain.ltd/categories*. For that you have to create a vhost on the server, which serves the content. This blog for instance points to a [Openshift's](https://www.openshift.com/) app, for which I created an alias. On [Koding](https://koding.com/) it is an stack domain. Don't forget to connect it to your virtual machine there. So vhosts may have different names, but you got the idea. 

## Hosted email service

You may want to send and receive emails with *master@mydomain.ltd*. Well, I have a good and a bad news for you. The bad first... big email hosters like Outlook and Gmail are in this game [no more](http://web.appstorm.net/roundups/email-roundups/the-best-places-to-host-your-email-with-your-own-domain/). The next one is Zoho and [claims to support *dot.tk* domain](https://forums.zoho.com/topic/add-dot-tk-domain-internal-error) but all i got is an error while trying to add my super cool domain name. I tried another ones like *25mail.st*, *inbox.com* or *mrmail* but either it confuses me or i can't create alias, forwarding or login with my email clients or the support is very bad.

Well the good news is - which i am very fond of - a hybrid method. I combine the power of sending emails for developers, like this blog sends me an email when i forgot my password, and the power of emails management like Gmail/Outlook. In fact it is Gmail/Outlook.

The reason I don't use a mail server on my Popo is the MX record cannot point to CNAME record. And well, I don't need to manage an mail server which can be down anytime.

### Mailgun

Even for nondevelopers [Mailgun](https://mailgun.com) is still a very good choice. Developers may read the API documentation.

After registering an account you can add *mydomain.ltd* to it. You will be provided with DNS records which you can add to your DNS provider. They are needed to send and receive secured emails.

Well, adding DNS record again requires up to 48 hours to take effect. Once available Mailgun will let you know on their site.

With no provision of mail boxes you need to take their routes functionality and a mailbox of Gmail/Outlook or whatever you want to manage your emails with. The only condition is the ability to send with other emails address (as alias maybe). Now, let's assume you have registered the email address *mydomain.ltd@gmail.com*.

With *Mailgun* you can send email with a sender like *noreply@mydomain.ltd*. Basically the part in front of *mydomain.ltd* can be anything, e.g. *nospam@blog.mydomain.ltd*. Or even *ihave@no.idea* if this domain isn't secured. The DNS record make sure only Mailgun can use your domain to send emails. For receiving emails sended to your certain address like *master@mydomain.ltd* you can use routes to forward these emails to *mydomain.ltd@gmail.com*. The routes can use regular expression, see the documentation for more information. It can look like this:

Priority: 10
Filter Expression: match_recipient("master@mydomain.ltd")
Action: forward("mydomain.ltd@gmail.com")
Description: i need to know who sends me messages!

The step above is needed to be done before you can register the *master@mydomain.ltd* as alias to send email with Gmail.

### Emails Management

I had two choices. Outlook is simple but powerful. It gives me free choice for my mail aliases. I chose it for my serious personal account. With Gmail you can just have an alias like *mydomain.ltd+alias@gmail.com* in order not to polute the name space. But I have Mailgun now.

You can go to Gmail, login, and click on settings (the wheel icon for that matter) at the right side. Under **Accounts and Import** you can **add another email address that you own**. If you add *mydomain.ltd+anything@gmail.com*, it will be your alias to use. In the case of *master@mydomain.ltd* you have to provide the information to send emails like: 

SMTP Hostname: smtp.mailgun.org
Default SMTP Login: postmaster@mydomain.ltd 
Default Password : `long hash code`
Ports: 25, 587 and 465 (TLS)

Which can be found under your domain in Mailgun. After that a validation email with a code will be sent to domain email address. Mailgun receives this and routes back to your Gmail address. Big round trip for a little email :). Well, weird thing is this email from Google will be in the spam folder. Find it there. Maybe you want to [create a filter](https://support.google.com/mail/answer/6579?hl=en) to add a label due to organization and to disallow the forwarding to the spam folder. Fill in the code and you are good to go. Don't forget to enable **Reply from the same address to which the message was sent** if you want to manage many email addresses from your domain with this Gmail account.

From the perpective of other people they just send to and receive from *master@mydomain.ltd* with no aware of your Gmail address. You can skip all these steps and use Zoho if you have a better ltd like com or org etc. But I am very happy with *hoangkim.tk* and Mailgun :).

## Last words

With money you can have all these at your provider. With less money you still can have a package which cries for perfection. The flexibility isn't garanteed in the 1st case either. And with no money... well, all you got is a ~~broken~~ decoupled system and works to do.