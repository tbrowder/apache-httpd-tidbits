Given:
+ a single Debian server
+ running Apache httpd
+ use as a mail server
+ use as a webserver with multiple virtual hosts
+ static IP address: 192.168.2.100
+ mail server name: *mail.example.com*

DNS records for all virtual hosts named *X.TLD* (including *example.com*):
```
X.TLD.      IN   A       w
WWW.X.TLD.  IN   CNAME   X.TLD.
@.          IN   MX      X.TLD.
@.          IN   TXT     "v=spf1 mx ?all"
X.TLD.      IN   MX      10 mail.example.com.
```

additional DNS records for *example.com*:
```
mail.example.com.           IN CNAME  example.com
100.2.168.192.in-addr.arpa. IN PTR    mail.example.com.
```


