---
title: "Chrome v58+ and self-signed certificates"
type: "post"
description: "How to properly generate self-signed certificates so that Chrome v58+ accepts them"
date: 2017-07-27
publishdate: 2017-06-03
showDate: true
tags: ["security", "linux", "ubuntu", "ssl", "certificates"]
categories: ["sysadmin"]
keywords: ["security", "linux", "ubuntu", "ssl", "certificates"]
clearReading: false
thumbnailImage: https://res.cloudinary.com/niceplace-github-io/image/upload/v1496551133/Chromev58andself-signedcertificates_edmbxz.png
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
---
This is a short guide to make self-signed HTTPS certificates work with locally hosted web applications and Google Chrome's browser.
<!--more-->

{{< alert warning >}}
Note : This has been tested on Ubuntu 16.04, some modifications may be needed to make it work on CentOS / MacOSX. Leave a suggestions in the comments if you had to change something I will update the post with it.
{{< /alert >}}

I'm using self signed certificates for my applications that I locally use (a [dockerized redmine](https://github.com/sameersbn/docker-redmine) in this case), not because I need the extra security but because I want to know how to configure it. Generating a certificate is rather easy and can be done with three short commands, given you have openssl installed.

{{< alert warning >}}
I'm showing the "old" and possibly invalid way of generating a self-signed certificate below for information purposes, the correct way to do it is shown at the end of the post.
{{< /alert >}}

{{< alert info >}}
Replace ***$KEY*** with the filename you wish to name your private key.  
Replace ***$CERTIFICATE_SIGNING_REQUEST*** with the filename you wish to name your certificate signing request.  
Replace ***$CERTIFICATE*** with the filename you wish to name your certificate.  
{{< /alert >}}

```
openssl genrsa -out $KEY.key 2048
openssl req -new -key $KEY.key -out $CERTIFICATE_SIGNING_REQUEST.csr
openssl x509 -req -days 365 -in redmine.csr -signkey $KEY.key -out $CERTIFICATE.crt

# This helps strengthen server security with stronger DHE parameters
# I have limited knowledge of what this means ;-)
openssl dhparam -out dhparam.pem 2048

```

However, Google Chrome recently changed the rules with regards to how they validate certificates and it seems to have broken my setup. It started with SHA-1 not being safe anymore, and now it appreas that Chrome no longer accepts self signed certificates that rely on "Common Name". I praise security for the world wide web but when it comes to local applications which I'm only securing "for fun", I wish there was a way I could just bypass the warning but hey, there's something more to learn !

It is worth mentionning that I had enabled HSTS in the nginx server bundled with my redmine installation which forces all connections to be HTTPS. If I had used Firefox I probably could have added an exception but HSTS does not allow me to just bypass the "this site is not secured" warning.

If you have an "old" self-signed certificate that is not valid in Chrome anymore, here is a solution you could try to fix the problem.

***Step 1 : Make sure that the old certificate is deleted from your Browser***  

In Chrome Settings -> Advanced Settings (expand it) -> HTTPS/SSL -> Manage Certificates...  
The, in the "Authorities" tab, look for the folder containing your local certificate (it was named "Personal" in my case) and delete it.

***Step 2 : Generate a certificate that will be valid with Chrome !***  
I found the solution [here](https://serverfault.com/questions/845766/generating-a-self-signed-cert-with-openssl-that-works-in-chrome-58) and I believe it was written for Mac OSX.

{{< alert info >}}
Here is an explanation for some of the parameters in the command below:  


***-keyout*** = the absolute/relative path for the private key file  
***-out*** = the absolute/relative path for the generated certificate  
***-subj*** = the hostname you want to use  
***-config*** = used to configure the subjectAltName parameter, you can use your hostname again
{{< /alert>}}

```
openssl req \
    -newkey rsa:2048 \
    -x509 \
    -nodes \
    -keyout server.key \
    -new \
    -out server.crt \
    -subj /CN=dev.mycompany.com \
    -reqexts SAN \
    -extensions SAN \
    -config <(cat /System/Library/OpenSSL/openssl.cnf \
        <(printf '[SAN]\nsubjectAltName=DNS:dev.mycompany.com')) \
    -sha256 \
    -days 3650
```


***Step 3 : Import the certificate in your local store***  
The certificate can be imported with the certutil command as shown below.
{{< alert warning >}}
If you do not have the certutil command, you can obtain it by installing the libnss3-tools package `sudo apt-get install libnss3-tools -y`
{{< /alert >}}

{{< alert info >}}
Replace ***$NICKNAME_FOR_CERT*** with a unique identifier to be attributed to your certificate  
Replace ***$PATH_TO_CERT*** with an absolute/relative path towards your self signed certificate  
***"P,,"*** means to add the certificate in the Personal store
{{< /alert >}}

`certutil -d sql:$HOME/.pki/nssdb -A -t "P,," -n $NICKNAME_FOR_CERT -i $PATH_TO_CERT`

Once I completed all these steps, I restarted my computer just to make sure everything was correctly updated (this habit is a courtesy of using Windows) and it worked ! The errors in Chrome were gone.
