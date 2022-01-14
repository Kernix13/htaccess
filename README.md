# htaccess Code Snippets

htaccess code snippets from my research on how to:

1. Secure WordPress
2. Speed up WordPress
3. And some miscelleaneous stuff

I know some of these are accurate but I'm not positive on the effectiveness or quality of the commands - use at your own discretion! Do your own research to double-check what I have here.

1. [Code for Security](#code-for-security)
2. [Code for Speed](#code-for-speed)
3. [Miscellaneous Code](#miscellaneous-code)
4. [Contributing](#contributing)

## WordPress Installs

The first code snippet is automatically added when you install WordPress. However, use the code blow that instaed of using a plugin for the scenarios listed.

Default code for single installs of WordPress:

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

For WordPress multi-site installs, there are 2 versions of the htaccess rules: 1. Subfolder example, 2. SubDomain example. Go to [WordPress htaccess Multisite](https://wordpress.org/support/article/htaccess/#multisite) for the code.

If WordPress is installed in some subdirectory, WordPress creates & uses the following .htaccess directives (subdirectory here is `/wordpress/`):

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

### Code for Security 

My goal for this section is to do as much security As I can myself and then look for a slim plugin to cover the rest. I know there are additional security snippets that you can add into other files like `wp-config.php`. The question is: **How can I find out everything that Wordfence does?** Then how do I find a plugin that covers the areas that the code below doesn't cover?

The next 70 lines, down to hot linking. Also code for protecting important files. What about 'Allow from sx.xxx.xxx.xxx' for all your IPs? Choose to limit login attempts in those cases. for security involving php.ini or php5.ini:

```apacheconf
<FilesMatch "^.*(error_log|wp-config\.php|php.ini|\.[hH][tT][aApP].*)$">
Order deny,allow
Deny from all
</FilesMatch>
```

Secure htaccess itself: one site I found has `allow`, `deny` (different order) and `satisfy all` instead of `deny all`:

```apacheconf
<files ~ "^.*\.([Hh][Tt][Aa])">
order allow,deny
deny from all
satisfy all
</files>
```

Secure admin area:

```apacheconf
<Files wp-login.php>
order deny,allow
Deny from all
Allow from xx.xxx.xxx.xxx
</Files>
```

Another one just for config file:

```apacheconf
<files wp-config.php>
order allow,deny
deny from all
</files>
```

What is this for? Not sure:

```apacheconf
Order deny,allow
Deny from all
<Files ~ ".(xml|css|jpe?g|png|gif|js)$">
Allow from all
</Files>
```

Deny Access To Certain Files (example):

```apacheconf
<files your-file-name.txt>
order allow,deny
deny from all
</files>
```

Restrict logins and admin access:

```apacheconf
# Limit logins and admin by IP
<Limit GET POST PUT>
order deny,allow
deny from all
allow from xx.xx.xx.xx
</Limit>
```

Edit `path-to-your-site` and `IP Address One$` and `$Two`, etc. with the actual IP addresses you want to have access to these pages:

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
To prevent directory browsing use: `Options All -Indexes`

Restrict access to php files:

```apacheconf
RewriteCond %{REQUEST_URI} !^/wp-content/plugins/file/to/exclude\.php
RewriteCond %{REQUEST_URI} !^/wp-content/plugins/directory/to/exclude/
RewriteRule wp-content/plugins/(.*\.php)$ - [R=404,L]
RewriteCond %{REQUEST_URI} !^/wp-content/themes/file/to/exclude\.php
RewriteCond %{REQUEST_URI} !^/wp-content/themes/directory/to/exclude/
RewriteRule wp-content/themes/(.*\.php)$ - [R=404,L]
```
Restrict PHP File Execution (hopefully not WordPress PHP files):

```apacheconf
<Files "*.php">
Order Deny,Allow
Deny from All
</Files>
</Directory>
```

Protect Against Script Injections:

```apacheconf
Options +FollowSymLinks
RewriteEngine On
RewriteCond %{QUERY_STRING} (<|%3C).*script.*(>|%3E) [NC,OR]
RewriteCond %{QUERY_STRING} GLOBALS(=|[|%[0-9A-Z]{0,2}) [OR]
RewriteCond %{QUERY_STRING} _REQUEST(=|[|%[0-9A-Z]{0,2})
RewriteRule ^(.*)$ index.php [F,L]
```

Securing the wp-includes Directory:

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
## Code for Speed

Right now I only have two categories of snippets here: 1. EXPIRES HEADER CACHING, 2. Enable GZIP. Are there any other snippets that assist with page load and speed?

Enable Browser Cache (double-check these values). I'm nt sure if these help with speed or if they are  just best practices that browsers want you to set. Perhaps both. The more "unique" ones from [WP Rocket Browser Caching](https://docs.wp-rocket.me/article/80-browser-caching):

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

Enable GZIP:

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

## Miscellaneous Code

Remove browser bugs (only needed for really old browsers):

```apacheconf
BrowserMatch ^Mozilla/4 gzip-only-text/html
BrowserMatch ^Mozilla/4\.0[678] no-gzip
BrowserMatch \bMSIE !no-gzip !gzip-only-text/html
Header append Vary User-Agent

</IfModule>
```

301 Redirect syntax (two versions:

```apacheconf
Redirect 301 /oldpage.html http://www.yoursite.com/newpage.html
Redirect 301 /oldpage2.html http://www.yoursite.com/folder/

RedirectMatch 301 /oldpage.html http://www.yoursite.com/newpage.html
RedirectMatch 301 /oldpage2.html http://www.yoursite.com/folder/
```

Here are 4 versions for hiding the extension of your pages (`.html` or `.php`). I tried a couple of these for my portfolio page, which is an HTML file, that I loaded into my `public_html` folder alongside my WordPress install. Nothing I did worked so I need to research these in particular. I didn't want to mess with it too much because WordPress is using the same `.htaccess` file:

```apacheconf
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}\.html -f
RewriteRule ^(.*)$ $1.html
```

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^([^\.]+)$ $1.html [NC, L]
```

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^([^\.]+)$ $1.html [NC,L] 
RewriteRule ^([^\.]+)$ $1.php [NC,L] 
```

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^([^/]+)/$ $1.html
RewriteRule ^([^/]+)/([^/]+)/$ /$1/$2.html
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} !(\.[a-zA-Z0-9]{1,5}|/)$
RewriteRule (.*)$ /$1/ [R=301,L]
```
Default language and charset though I do not think this is necessary:

```apacheconf
DefaultLanguage en
AddDefaultCharset UTF-8
```

## Contributing

Please open an issue if you know better htaccess code snippets than are listed here, or if you want to add better desriptions and notes. I would like to make this readme file a go-to resource for developers.
