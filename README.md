# htaccess

htaccess code snippets from my research on how to:

1. Speed up WordPress
2. Secure WordPress
3. And some miscelleaneous stuff

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

default language and char set 

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

Restrict admin access, edit 'path-to-your-site', also edit IP Address One$...with the actual IP addresses you want to have access to these pages

```apacheconf
# Limit logins and admin by IP
<Limit GET POST PUT>
order deny,allow
deny from all
allow from xx.xx.xx.xx
</Limit>
```

do these lines go with the bloack above?

```apacheconf
ErrorDocument 401 /path-to-your-site/index.php?error=404
ErrorDocument 403 /path-to-your-site/index.php?error=404

<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{REQUEST_URI} ^(.*)?wp-login\.php(.*)$ [OR]
RewriteCond %{REQUEST_URI} ^(.*)?wp-admin$
RewriteCond %{REMOTE_ADDR} !^IP Address One$
RewriteCond %{REMOTE_ADDR} !^IP Address Two$
RewriteCond %{REMOTE_ADDR} !^IP Address Three$
RewriteRule ^(.*)$ - [R=403,L]
</IfModule>
```
Prevent directory browsing: `Options All -Indexes`

Restrict access to php files

```apacheconf
RewriteCond %{REQUEST_URI} !^/wp-content/plugins/file/to/exclude\.php
RewriteCond %{REQUEST_URI} !^/wp-content/plugins/directory/to/exclude/
RewriteRule wp-content/plugins/(.*\.php)$ - [R=404,L]
RewriteCond %{REQUEST_URI} !^/wp-content/themes/file/to/exclude\.php
RewriteCond %{REQUEST_URI} !^/wp-content/themes/directory/to/exclude/
RewriteRule wp-content/themes/(.*\.php)$ - [R=404,L]
```
Restrict PHP File Execution (hopefully not WordPress PHP files)

```apacheconf
<Files "*.php">
Order Deny,Allow
Deny from All
</Files>
</Directory>
```

Protect Against Script Injections

```apacheconf
Options +FollowSymLinks
RewriteEngine On
RewriteCond %{QUERY_STRING} (<|%3C).*script.*(>|%3E) [NC,OR]
RewriteCond %{QUERY_STRING} GLOBALS(=|[|%[0-9A-Z]{0,2}) [OR]
RewriteCond %{QUERY_STRING} _REQUEST(=|[|%[0-9A-Z]{0,2})
RewriteRule ^(.*)$ index.php [F,L]
```

Securing the wp-includes Directory

```apacheconf
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^wp-admin/includes/ - [F,L]
RewriteRule !^wp-includes/ - [S=3]
RewriteRule ^wp-includes/[^/]+\.php$ - [F,L]
RewriteRule ^wp-includes/js/tinymce/langs/.+\.php - [F,L]
RewriteRule ^wp-includes/theme-compat/ - [F,L]
</IfModule>
```

Prevent Username Enumeration: 
```apacheconf
RewriteCond %{QUERY_STRING} author=d
RewriteRule ^ /? [L,R=301]
```

Enable Browser Cache (Double-check these values):

```apacheconf

```

ss

```apacheconf

```

ss

```apacheconf

```

ss

```apacheconf

```

ss

```apacheconf

```

ss

```apacheconf

```

ss

```apacheconf

```

