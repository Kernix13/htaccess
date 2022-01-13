# htaccess

Code from my research on how to:

1. Speed up WordPress
2. Secure WordPress

Default code for single installs of WordPress - Security

```apacheconf
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

If WordPress is installed in some subdirectory, WordPress creates & uses the following .htaccess directives (subdirectory here is /wordpress/)

```apacheconf
# BEGIN WordPress	
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /wordpress/
# /Blog/ for me
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /wordpress/index.php [L]
</IfModule>
# END WordPress
```

# default language and char set 

```apacheconf
DefaultLanguage en
AddDefaultCharset UTF-8
```

> Security next 70 lines, down to hot linking
>	Protect important files
>	Could not find php.ini or php5.ini so remove that one
>	what about 'Allow from sx.xxx.xxx.xxx' for all your IPs? choose to limit login attempts in thse cases

```apacheconf
<FilesMatch "^.*(error_log|wp-config\.php|php.ini|\.[hH][tT][aApP].*)$">
Order deny,allow
Deny from all
</FilesMatch>
```

secure htaccess itslf, one site has allow, deny (diff order) and satisfy all instead of deny all

```apacheconf
<files ~ "^.*\.([Hh][Tt][Aa])">
order allow,deny
deny from all
satisfy all
</files>
```

secure admin area

```apacheconf
<Files wp-login.php>
order deny,allow
Deny from all
Allow from xx.xxx.xxx.xxx
</Files>
```

another one just for config?

```apacheconf
<files wp-config.php>
order allow,deny
deny from all
</files>
```

What is this for?

```apacheconf
Order deny,allow
Deny from all
<Files ~ ".(xml|css|jpe?g|png|gif|js)$">
Allow from all
</Files>
```

Deny Access To Certain Files

```apacheconf
<files your-file-name.txt>
order allow,deny
deny from all
</files>
```




