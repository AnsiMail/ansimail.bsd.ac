+++
title = "Detailed installation"
author = ["Aisha Tammy"]
lastmod = 2020-06-16T15:48:45-04:00
draft = false
weight = 1009
+++

## Assumptions {#assumptions}

This is a install guide for the common human, who has just learned that she wants to set up and email server.
She know nothing about anything related to setting up a mail server and only has a domain name and couple of bucks (<$10) in her hand.

We will use those $10 to do some really cool things:

-   Buy a small VPS (with both ipv4 and ipv6)
-   Buy a secondary DNS provider
-   Set up Excision on the server


## General Information {#general-information}

Hosting your own email is not going to be free.
I'm just putting it out there for our younger enthusiasts and students who want to try out things in their home without shelling out any money.

BUT, that was kind of a lie.
You could host the email in your house but it is going to require certain considerations. I will mention those in places where appropriate, about what steps to take if you are working at home. Most of which is going to be related to nameserver configurations. ****But even with all of that, if you host it from home, you will not be able to use it for anything other than testing and learning, as all home ip addresses are banned from sending out email, as a precaution against virus infected computers which perpetually keep sending spam****.


### Getting a static ip (for noobs like me) {#getting-a-static-ip--for-noobs-like-me}

One of the first things that you need to understand is that you are going to need a static IP. The easiest way to get this is to use a VPS hosting provider, who also supports OpenBSD, one of the better ones is [Vultr](<https://vultr.com>). For a list of recommended providers, take a look at the [github issue](https://github.com/Excision-Mail/Excision-Mail/issues/19).

You can select the lowest plan for $5 per month, which gives you the enough computing power to try out Excision without turning off any features :)
The smaller ones are a bit too small for anything worthwhile, imho.

You server requirements:


### Now that we have a server {#now-that-we-have-a-server}

I am also assuming that you have a domain to your name and that you hold the power to configure everything related to that domain. This includes access to the domain registrar for setting primary nameservers, access to a horrible web-ui for doing "AdVanCEd" DNS condigurations and all that.
The first thing that we do is to start making preparations for our DNS setup, which we want to manage ourself.


## Prerequisites {#prerequisites}

If you skipped to this section, the prerequisites are:

-   OpenBSD 6.6 host with static ipv4 and ipv6, which also allows setting up reverse DNS
-   Access to domain registrar for setting nameservers


### Set up secondary nameservers {#set-up-secondary-nameservers}

One of the first things that we are going to do is to get a secondary nameserver service.
Excision comes with a automated stealth master NSD configuration using the default NSD service in OpenBSD.
The advantage of this is to be able to modify complex DNS records easily via text configuration which is nicely documented, explaining each option. If anybody has ever tried to work with a web ui based dns configuration and tried to set SRV records, they will know how insanely tedious and complicated it really is.
Thankfully the worst part of the DNS configuration is automated leaving you with almost nothing to manage yourself (though you can if you want to).

For a secondary nameserver, the minimum requirements are to be able to accept NOTIFY (which informs the secondary about any updates from your computer).
Look at the pinned issue for a recommended list of secondary providers. Most services are really cheap at < $2 per month, for more than 10 domains at a time. So if you have a friend it is useful to do this together, as Excision also supports multiple domain email handling.

The secondary DNS provider will give you two kinds of ip lists

-   **public nameservers**: These are the servers that other people on the internet will think are the primary nameservers of your domain. They will not know about the master DNS resolvers running on your computer (hence stealth master). Most probably each nameserver will have a name (like ns7.provider.tld), an ipv4 and an ipv6. Note these down because they are needed to generate the configuration file. Also go to your domain registrar and register each of the public nameservers as your primary nameservers.
-   **secondary nameservers**: To find the nameserver ip addresses you might need to look around a bit and poke the buttons on the providers api. Note these down as well because they too are needed to generate the configuration file.

These two are the longest configuration options and everything after this is smooth sailing.


## Set up variables file {#set-up-variables-file}

The configuration file for Excision is called \`vars.yml\` which is supposed to be the filled-in version of the \`vars-sample.yml\` file.
Read the \`vars-sample.yml\` file in depth because all the options have been explained in great detail, so please make sure that you understand each of them.

You will see that you need to enter the two lists of ip addresses in the two options provided for the stealth master configuration to work.

First step that you need to do is to make sure that your system is bootstrapped correctly, to get ansible working.

The assumption going forwarded is that you have downloaded and extracted Excision to some directory and it is the current working directory.

{{< highlight sh "linenos=table, linenostart=1" >}}
sh scripts/bootstrap.sh
{{< /highlight >}}

This installs the necessary packages, Ansible and GnuPG on your server.
(Currently GnuPG is to be installed manually because it cannot be installed through Ansible due to package ambiguity)


## Run preinstallation playbook {#run-preinstallation-playbook}

After the system finishes bootstrapping you need to run the first playbook: \`site-preinstall.yml\`

{{< highlight sh "linenos=table, linenostart=1" >}}
ansible-playbook site-preinstall.yml
{{< /highlight >}}

This is going to take a while because it installs quite a bit of packages, so I suggest going and getting some Kombucha.

Also after running this playbook it is advisable to wait a couple of minutes for the site updates to propogate through the interwebs and letting your secondary nameservers update their configurations. Because even though they do accept NOTIFY, I have found that certain servers take some time to update the configuration. Generally 5-10 minuts is enough.


## Run full installation playbook {#run-full-installation-playbook}

Now that everyone on the webz knows about your new server names and services, it is time to install the full playbook:

{{< highlight sh "linenos=table, linenostart=1" >}}
ansible-playbook site-install.yml
{{< /highlight >}}

After this finishes running you should reboot your server to make sure that all the services are going to be using the proper configurations.

AND YOU ARE DONE!

Excision has finished installing on your system and you have a working mail server (which you are unable to access because you don't know the password of your email account :P)


## Post ansible finishing steps {#post-ansible-finishing-steps}

Now that the server has been rebooted and Excision is running, you need to reset the password of you admin account:

Supposing that your adminstrator is called \`notaisha\` and your domain was \`aisha.cc\`, run the following command to change the password and reload the services

{{< highlight sh "linenos=table, linenostart=1" >}}
excision change-passwd "notaisha@aisha.cc"
excision virtual-regen
{{< /highlight >}}

You can read the github wiki for some general purpose server maintenance commands that Excision adds to the system. They are supremely helpful :)


### Testing your email {#testing-your-email}

Now that you know your email address and password, its time to test out the shiny new email while it still has that new-email smell.

There is no web-mail configured yet (it is going to be soon), you need to use an email client to access this server.

Some recommended email clients are:

-   Thunderbird
-   KMail
-   Evolution
-   mutt/neomutt
-   Literally anything in the world, Excision is configured to make everyone auto-detect all ports and domain settings automatically

Your username is \`<admin>@<domain.tld>\`, where you fill your own credentials and your password is what you set in the previous step.

Try sending mails to some other accounts and see if they reach correctly.

A good test is to go on <https://mail-tester.com> and see what score you get. You should see a 10/10, cuz this setup is fire.

Don't hesitate to ask any questions on IRC or github. I might not be able to respond immediately but I will try to be fast.

Take care, be safe and get back your privacy from Big Brother :)
