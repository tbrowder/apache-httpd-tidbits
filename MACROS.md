# An Apachec httpd macro for https hosts

## Include in an httpd conf file

The example macro is named **VHOSTHTTPS** and has inputs of **${PROJECT}** and **${TLD}**.
Following the macro's definition we use the macro three times to define three virtual hosts.
As far as I know, we could define a practically unlimited number of virtual hosts.

Currently I have 12 virtual hosts on line using this macro.


```
<Macro VHOSTHTTPS ${PROJECT} ${TLD}>
  <VirtualHost *:80>
    ServerName  ${PROJECT}.${TLD}
    ServerAlias www.${PROJECT}.${TLD}
    # enforce ssl/tls
    Redirect Permanent / https://${PROJECT}.${TLD}/
  </VirtualHost>

  ##### BEGIN TLS OPERATIONS #####
  <VirtualHost *:443>
    SSLEngine on
    ServerName  ${PROJECT}.${TLD}
    ServerAlias www.${PROJECT}.${TLD}
    DocumentRoot /home/web-server/${PROJECT}.${TLD}/public
    <Directory /home/web-server/${PROJECT}.${TLD}/public>
      SSLRequireSSL
    </Directory>

    # cgi directory
    ScriptAlias     /cgi-bin/       /home/web-server/${PROJECT}.${TLD}/cgi-bin
    # data directory
    Alias           /data/          /home/web-server/${PROJECT}.${TLD}/data

    # server SSL/TLS certificate data
    SSLCertificateFile      /etc/letsencrypt/live/${PROJECT}.${TLD}/fullchain.pem
    SSLCertificateKeyFile   /etc/letsencrypt/live/${PROJECT}.${TLD}/privkey.pem

    # The following belongs in the virtual host server context inside
    # a Directory directive (may be a macro later).
    # THE PRIVATE AREA
    <Directory ~ ".*/private">

      #--- following varies for file or dbd
      AuthType basic
      AuthName ${PROJECT}.${TLD}
      AuthBasicProvider dbm
      AuthDBMType DB
      AuthDBMUserFile /home/web-server/passwords/${PROJECT}.${TLD}.dbm

      Require valid-user

      # may be problems with session as I am using it
      # 60 min max
      SessionMaxAge 3600

    </Directory>

    # The following is the login handler, the login form needs to
    # point to this handler in its action!
    <Location /dologin>
      SetHandler form-login-handler
      AuthFormLoginRequiredLocation https://${PROJECT}.${TLD}/login.html
      AuthFormLoginSuccessLocation https://${PROJECT}.${TLD}/private/index.html

      #--- following varies for file or dbd
      AuthType form
      AuthName ${PROJECT}.${TLD}
      AuthFormProvider dbm
      AuthDBMType DB
      AuthDBMUserFile /home/web-server/passwords/${PROJECT}.${TLD}.dbm

      Session On
      SessionCookieName session path=/

      # The following requires mod_crypto, pass phrase considerations
      # (use Last Pass to generate a strong one and put it in a file).
      SessionCryptoPassphraseFile /home/web-server/passwords/${PROJECT}.${TLD}.secret
    </Location>

    # This is the location setting: When a user comes to this location
    # unauthorized, he will be redirected to the login form This happens
    # as the ErrorDocument gets overwritten with the login page.
    <Location /private/index.html>
      ErrorDocument 401 /login.html

      #--- following varies for file or dbd
      AuthType form
      AuthName ${PROJECT}.${TLD}
      AuthFormProvider dbm

      AuthFormLoginRequiredLocation https://${PROJECT}.${TLD}/login.html

      Require valid-user

      Session On
      SessionCookieName session path=/

      # The following requires mod_crypto, pass phrase considerations
      # (use Last Pass to generate a strong one and put it in a file).
      SessionCryptoPassphraseFile /home/web-server/passwords/${PROJECT}.${TLD}.secret
    </Location>

  </VirtualHost>
  ##### END TLS OPERATIONS #####

</Macro>

# use the macro as many times as needed
#                          PROJECT             TLD
Use VHOSTHTTPS             example             us
Use VHOSTHTTPS             example             com
Use VHOSTHTTPS             example2            org

# undefine the macro after use
UndefMacro VHOSTHTTPS
```
