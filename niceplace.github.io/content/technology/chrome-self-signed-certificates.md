---
title: "Chrome v58+ and self-signed certificates"
type: "post"
description: "How to properly generate self-signed certificates so that Chrome v58+ accepts them"
date: 2017-05-27
publishdate: 2017-05-27
tags: ["security", "linux", "ubuntu", "ssl", "certificates"]
categories: ["sysadmin"]
keywords: ["security", "linux", "ubuntu", "ssl", "certificates"]
clearReading: false
thumbnailImage: ../images/Chromev58andself-signedcertificates.png
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
showDate: true
---
A short and simple guide to get self-signed HTTPS certificates to work with locally hosted web applications and Google Chrome's browser.
<!--more-->

Note : This has been tested on Ubuntu 16.04, some modifications may be needed to make it work on CentOS / MacOSX. Leave a suggestions in the comments if you had to change something I will update the post with it.

I'm using self signed certificates for my applications that I locally use, not because I need the extra security but because I want to know how to basically configure it. Generating a certificate is rather easy and can be done with three short commands, given you have openssl installed.

Replace $KEY with the filename you wish to name your private key
Replace $CERTIFICATE_SIGNING_REQUEST with the filename you wish to name your certificate signing request
Replace $CERTIFICATE with the filename you wish to name your certificate

```
openssl genrsa -out $KEY.key 2048
openssl req -new -key $KEY.key -out $CERTIFICATE_SIGNING_REQUEST.csr
openssl x509 -req -days 365 -in redmine.csr -signkey $KEY.key -out $CERTIFICATE.crt

# This helps strengthen server security with stronger DHE parameters
# I have limited knowledge of what this means ;-)
openssl dhparam -out dhparam.pem 2048

```

However, Google Chrome recently changed the rules with regards to how they validate certificates and it seems to have broken my setup. It started with SHA-1 not being safe anymore, and now it appreas that Chrome no longer accepts self signed certificates that rely on "Common Name". I praise security for the world wide web but when it comes to local applications which I'm only securing "for fun", I wish there was a way I could just bypass the warning but hey, there's something more to learn !

It is worth mentionning that I had enabled HSTS in the nginx server bundled with my redmine installation which forces all connections to be HTTPS. If I had used Firefox I probably could have added an exception in other cases but HSTS is not compatible with that.

If you have an "old" self-signed certificate that is not valid in Chrome anymore, here is a solution you could try to fix the problem.

Step 1 : Make sure that the old certificate is deleted from your Browser
In Chrome Settings -> Advanced Settings (expand it) -> HTTPS/SSL -> Manage Certificates...
In the "Authorities" tab, look for the folder containing your local certificate (it was named "Personal" in my case) and delete it.

Step 2 : Generate a certificate that will be valid with Chrome ! I found the solution [here](https://serverfault.com/questions/845766/generating-a-self-signed-cert-with-openssl-that-works-in-chrome-58) and I believe it was made on a Mac.

Here is an explanation for some of the parameters

-keyout = the absolute/relative path for the private key file
-out = the absolute/relative path for the generated certificate
-subj = the hostname you want to use
-config = used to configure the subjectAltName parameter, use your hostname again

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


Step 3 : Import the certificate in your local store
Note : if you do not have the certutil command, you can obtain it by installing the libnss3-tools package `sudo apt-get install libnss3-tools -y`

$NICKNAME_FOR_CERT represents a unique identifier to be attributed to your certificate
$PATH_TO_CERT the absolute/relative path towards your self signed certificate
"P,," means to add the certificate in the Personal store


`certutil -d sql:$HOME/.pki/nssdb -A -t "P,," -n $NICKNAME_FOR_CERT -i $PATH_TO_CERT`

I restarted my computer just to make sure everything was correctly updated (this habit is a courtesy of using Windows) and it worked ! The errors in Chrome were gone.
