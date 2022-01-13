# htaccess

htaccess code snippets from my research on how to:

1. Speed up WordPress
2. Secure WordPress
3. And some miscelleaneous stuff

Default code for single installs of WordPresS
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

Enable Browser Cache (Double-check these values). More exotic ones from [WP Rocket Browser Caching](https://docs.wp-rocket.me/article/80-browser-caching):

```apacheconf
## EXPIRES HEADER CACHING ##
<IfModule mod_expires.c>
ExpiresActive On

# MEDIA
ExpiresByType image/jpg   "access 1 year"
ExpiresByType image/jpeg  "access 1 year"
ExpiresByType image/gif   "access 1 year"
ExpiresByType image/png   "access 1 year"
ExpiresByType image/svg   "access 1 year"
ExpiresByType image/webp  "access plus 4 months"
ExpiresByType image/avif  "access plus 4 months"
ExpiresByType video/ogg   "access plus 4 months"
ExpiresByType audio/ogg   "access plus 4 months"
ExpiresByType video/mp4   "access plus 4 months"
ExpiresByType video/webm  "access plus 4 months"

# Favicon (cannot be renamed!)
ExpiresByType image/x-icon "access 1 year"

ExpiresByType image/avif-sequence   "access plus 4 months"
ExpiresByType image/svg+xml         "access plus 1 month"

# HTML, CSS
ExpiresByType text/html   "access 0 seconds"
ExpiresByType text/css    "access 1 month"

# PDF
ExpiresByType application/pdf         "access 1 month"

# JAVASCRIPT
ExpiresByType application/javascript  "access 1 month"

# HTML components (HTCs)
ExpiresByType text/x-component  "access plus 1 month"

# Manifest files
ExpiresByType text/cache-manifest                   "access plus 0 seconds"
ExpiresByType application/x-web-app-manifest+json   "access plus 0 seconds"

# Data interchange
ExpiresByType text/xml                "access plus 0 seconds"
ExpiresByType application/json        "access plus 0 seconds"
ExpiresByType application/xml         "access plus 0 seconds"

# Web feeds
ExpiresByType application/rss+xml     "access plus 1 hour"
ExpiresByType application/atom+xml    "access plus 1 hour"

# Web fonts
ExpiresByType font/ttf    "access plus 4 months"
ExpiresByType font/otf    "access plus 4 months"
ExpiresByType font/woff   "access plus 4 months"
ExpiresByType font/woff2  "access plus 4 months"
ExpiresByType application/vnd.ms-fontobject "access plus 1 month"

ExpiresDefault "access 3 days"
</IfModule>
## EXPIRES HEADER CACHING ##
```

Enable GZIP

```apacheconf
<IfModule mod_deflate.c>

# Compress HTML, CSS, JavaScript, Text, XML and fonts - what is this?
AddOutputFilterByType DEFLATE application/javascript
AddOutputFilterByType DEFLATE application/rss+xml
AddOutputFilterByType DEFLATE application/vnd.ms-fontobject
AddOutputFilterByType DEFLATE application/x-font
AddOutputFilterByType DEFLATE application/x-font-opentype
AddOutputFilterByType DEFLATE application/x-font-otf
AddOutputFilterByType DEFLATE application/x-font-truetype
AddOutputFilterByType DEFLATE application/x-font-ttf
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE font/opentype
AddOutputFilterByType DEFLATE font/otf
AddOutputFilterByType DEFLATE font/ttf
AddOutputFilterByType DEFLATE image/svg+xml
AddOutputFilterByType DEFLATE image/x-icon
AddOutputFilterByType DEFLATE text/css
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/xml
```

Remove browser bugs (only needed for really old browsers)

```apacheconf
BrowserMatch ^Mozilla/4 gzip-only-text/html
BrowserMatch ^Mozilla/4\.0[678] no-gzip
BrowserMatch \bMSIE !no-gzip !gzip-only-text/html
Header append Vary User-Agent

</IfModule>
```
