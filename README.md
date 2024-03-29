# htaccess Code Snippets

htaccess code snippets from my research on how to:

1. Secure WordPress
2. Speed up WordPress
3. And some miscelleaneous stuff

I know some of these are accurate but I'm not positive on the effectiveness or quality of the commands - use at your own discretion! Do your own research to double-check what I have here. Some of  the snippets are WordPress specific, others are general purpose.

Some of the snippets are short so I am not adding links to each individual code block, but you will see them nested under the main category.

<div id="back-to-top"></div>

## Table of Contents

1. [WordPress Installs](#wordpress-installs)
   1. [Single install](#single-install)
   1. [Subdirectory install](#subdirectory-install)
1. [Code for Security](#code-for-security)
   1. Limit login attempts
   1. Secure the htaccess file
   1. Secure admin area
   1. Secure wp-config.php
   1. Deny specific file access
   1. Restrict logins and admin access
   1. Prevent directory browsing
   1. Restrict access to php files
   1. Restrict PHP File Execution
   1. Protect Against Script Injections
   1. Securing the wp-includes Directory
   1. Prevent Username Enumeration
1. [Code for Speed](#code-for-speed)
   1. [Enable Browser Cache](#enable-browser-cache)
   1. [Enable GZIP](#enable-gzip)
1. [Miscellaneous Code](#miscellaneous-code)
   1. Remove browser bugs
   1. [301 Redirect syntax](#301-redirect-syntax)
   1. [Hide page extensions](#hide-page-extensions)
   1. [Default language and charset](#default-language-and-charset)
1. [Woocommerce](#woocommerce)
   1. [Redirect syntax](#redirect-syntax)
   1. [Bluehost Nameservers](#bluehost-nameservers)
   1. [The deactivation code](#the-deactivation-code)
   1. [MySQL queries to check for Woocommerce fields](#mysql-queries-to-check-for-woocommerce-fields)
   1. [IMPORTANT NOTE](#important-note)
1. [Contributing](#contributing)

## WordPress Installs

The first code snippet is automatically added when you install WordPress. However, the code blocks after the basic install block is for installing WordPress on a subdomain or in subfolders. I also have a link for the code for WordPress multi-sites. Don't use a plugin if you do not need to for these scenarios

### Single install 

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
- - - 

For WordPress multi-site installs, there are 2 versions of the htaccess rules: 1. Subfolder example, 2. SubDomain example. Go to [WordPress htaccess Multisite](https://wordpress.org/support/article/htaccess/#multisite) for the code.

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### Subdirectory install

If WordPress is installed in a subdirectory, WordPress creates & uses the following .htaccess directives (subdirectory here is `/wordpress/`):

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

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

## Code for Security 

My goal for this section is to do as much security as I can myself and then look for a slim plugin to cover the rest. I know there are additional security snippets that you can add into other files like `wp-config.php`. The question is: **How can I find out everything that Wordfence does?** Then how do I find a plugin that covers the areas that the code below doesn't cover?

> What about 'Allow from sx.xxx.xxx.xxx' for all your IPs? 

Choose to limit login attempts in those cases. for security involving `php.ini` or `php5.ini`:

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

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

Deny access to certain files (example):

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
To prevent directory browsing by hiding WordPress directions use: `Options All -Indexes`

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

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

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

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

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

## Code for Speed

Right now I only have two categories of snippets here: 1. EXPIRES HEADER CACHING, 2. Enable GZIP. Other things that may help are:

- Rule for setting `Cache-Control` headers
- Activate the Keep Alive resource
- Prevent image hotlinking

### Enable Browser Cache

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
ExpiresByType image/x-icon  "access 1 year"
ExpiresByType image/svg+xml "access plus 1 month"
ExpiresByType audio/ogg   "access plus 4 months"
ExpiresByType video/ogg   "access plus 4 months"
ExpiresByType video/mp4   "access plus 4 months"
ExpiresByType video/webm  "access plus 4 months"

# HTML, CSS, HTML components (HTCs)
ExpiresByType text/html   "access 0 seconds"
ExpiresByType text/css    "access 1 month"
ExpiresByType text/x-component  "access plus 1 month"

# PDF
ExpiresByType application/pdf   "access 1 month"

# JAVASCRIPT
ExpiresByType application/javascript  "access 1 month"

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
## END EXPIRES HEADER CACHING ##
```

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### Enable GZIP

```apacheconf
# BEGIN Gzip
<IfModule mod_deflate.c>
AddType x-font/woff .woff
AddType x-font/ttf .ttf
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
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE x-font/ttf
AddOutputFilterByType DEFLATE application/vnd.ms-fontobject
AddOutputFilterByType DEFLATE font/opentype font/ttf font/eot font/otf
</IfModule>
# END Gzip
```

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

## Miscellaneous Code

Some other options other than what you see below are:
- Remove the slash "/" at the end of the URL
- Create a custom 404 Error page

Remove browser bugs only needed for really old browsers):

```apacheconf
BrowserMatch ^Mozilla/4 gzip-only-text/html
BrowserMatch ^Mozilla/4\.0[678] no-gzip
BrowserMatch \bMSIE !no-gzip !gzip-only-text/html
Header append Vary User-Agent

</IfModule>
```

### 301 Redirect syntax

> (two versions):

```apacheconf
Redirect 301 /oldpage.html http://www.yoursite.com/newpage.html
Redirect 301 /oldpage2.html http://www.yoursite.com/folder/

RedirectMatch 301 /oldpage.html http://www.yoursite.com/newpage.html
RedirectMatch 301 /oldpage2.html http://www.yoursite.com/folder/
```

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### Hide page extensions

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

Here is the code that WordPress generates. Do I need `<IfModule mod_rewrite.c>` and `</IfModule>`?
```apacheconf
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
```

Here is what I tried after END WordPress and it did not work:
```apacheconf
# BEGIN custom rewrite for html files
<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}\.html -f
RewriteRule ^(.*)$ $1.html
</IfModule>
# END custom rewrite rule
```

### Default language and charset

> though I do not think this is necessary:

```apacheconf
DefaultLanguage en
AddDefaultCharset UTF-8
```

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

## Woocommerce

Woocommerce instructions for full removal of their plugin

> There are two things to understand when uninstalling or removing WooCommerce

- If you deactivate and delete the plugin from WordPress, you only remove the plugin and its files. Your settings, orders, products, pages, etc… will still exist in the database
- If you need to remove ALL WooCommerce data, including products, order data, etc., you need to be able to modify the site’s `wp-config.php` file before deactivating and deleting the plugin

> To fully remove all WooCommerce data from your WordPress site, open your site’s `wp-config.php` file. Scroll down until you fine the the bottom, add the following constant on its own line above the “That’s all, stop editing” comment.

```php
define( 'WC_REMOVE_ALL_DATA', true );
/* That’s all, stop editing! Happy publishing. */ 
```
** HUGE NOTE**: You have to have the plugin installed before you add the code to `wp-config.php`. Then when you deactivate the plugin and remove it the cod will run and remove everything (see bottom of this file).

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### Redirect syntax

Is Bluehost running on Apache or a different server?

This is the basic format from CSS Tricks [301 Redirects article](https://css-tricks.com/snippets/htaccess/301-redirects/)  (Apache servers only):

```apacheconf
Redirect 301 /oldpage.html http://www.yoursite.com/newpage.html
Redirect 301 /oldpage2.html http://www.yoursite.com/folder/
```

On Siteground I have the following examples where Siteground added the `RedirectMatch` redirects from the redirect tool in cPanel:

```apacheconf
RedirectMatch 301 /Blog/pet-services-blog/pet-business/online-business-listing-review-sites-pet-business/ /Blog/category/pet-services-blog/pet-business/
RedirectMatch 301 /oldpage2.html http://www.yoursite.com/folder/

Redirect 301 fairmountpetservice.com/index.html https://fairmountpetservice.com/Blog/index/
Redirect 301 /oldpage.html http://www.yoursite.com/newpage.html
```

**Note**: My redirects are placed directly above the comment:

```apacheconf
# BEGIN WordPress
```

And I have this commented line before the start of my redirects:

```apacheconf
## WOOCOMMERCE AND STRIPE MANUAL REDIRECTS ##
```

Here are example redirects for my pet service site (both versions), though I used `RedirectMatch`:

```apacheconf
Redirect 301 /cart/ https://fairmountpetservice.com/Blog/
RedirectMatch 301 /cart/ https://fairmountpetservice.com/Blog/

Redirect 301 /checkout/ https://fairmountpetservice.com/Blog/
RedirectMatch 301 /checkout/ https://fairmountpetservice.com/Blog/

Redirect 301 /my-account/ https://fairmountpetservice.com/Blog/
RedirectMatch 301 /my-account/ https://fairmountpetservice.com/Blog/

Redirect 301 /products/ https://fairmountpetservice.com/Blog/
RedirectMatch 301 /products/ https://fairmountpetservice.com/Blog/
```

I had one already done with `RedirectMatch`: 

```apacheconf
RedirectMatch 301 /shop/ https://fairmountpetservice.com/Blog/
```

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### Bluehost Nameservers

In case they are needed:

```
ns1.bluehost.com	162.88.60.37
ns2.bluehost.com	162.88.61.37
```

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### The deactivation code

Here is the [Woocommerce Github code](https://github.com/woocommerce/woocommerce/blob/master/uninstall.php) for the snippet added to `wp-config.php`.

Looks like the uninstall.php file is doing the following things:

1. Unschedules all events attached to woo hooks using wp_clear_scheduled_hook starting on line 15
2. Dropping Woocommerce admin tables using ::drop_tables() on line 31
3. Removing "roles + caps" using ::remove_roles() on line 36 
4. Trashing pages using wp_trash_post( get_option ( 'various woocommerce page id names' ) ) starting on line 39
5. doing something with attributes if there are tables prefixed with woocommerce_... starting on line 48
6. dropping woo tables again, this time I assume for the plugin as opposed to the admin area on line 55 
7. delete woo records in the options table lines 58 & 59
8. delete woo records in the usermeta table line 62 
9. delete posts and data (posts, postmeta, comments, commentsmeta) starting on line 65
10. conditional delete of taxonomies & attributes if > WP version 4.2, line 75 (2 lines directly below): 
11. foreach delete of term taxonomies line 77
12. foreach delete of term attributes line 87
13. delete orphan records, terms, and term meta line 96
14. wp_cache_flush line 109

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>

### MySQL queries to check for Woocommerce fields

SQL examples to find woocommerce fields in the `options` table:

```sql
SELECT * FROM `wpaa_options` WHERE option_name LIKE 'woocommerce\_%' OR option_name LIKE 'widget\_woocommerce\_%'
SELECT * FROM `wpaa_options` WHERE option_name LIKE 'woocommerce\_%'
SELECT * FROM `wpaa_options` WHERE option_name LIKE 'widget\_woocommerce\_%'
```

The last 2 SQL statements are just to run individual queries, but then you proabably know that if you know how to run queries in phpmyadmin. I had 127 records in total when I ran the combined query. Then I commented out the snippet in `wp-config.php`, installed Woo, un-commented the code in config, and finally deactivated and removed Woo. I ran the query again and it returned **ZERO** records.

Here is an example of SQL statements to manually delete the recor from the `options` table:

```sql
DELETE * FROM `wpaa_options` WHERE option_name LIKE 'woocommerce\_%' OR option_name LIKE 'widget\_woocommerce\_%'
DELETE * FROM `wpaa_options` WHERE option_name LIKE 'woocommerce\_%'
DELETE * FROM `wpaa_options` WHERE option_name LIKE 'widget\_woocommerce\_%'
```

So the removal process ran the code snippet in the config file most likely because it is hooking onto the deactivation and removal of the plugin. If you remove the plugin before adding the coee snippet then there is nothing to hook onto.

### IMPORTANT NOTE

**IF YOU INTENT TO INSTALL WOOCOMMERCE AFTER DOING THIS REMOVAL PROCESS, YOU WILL HAVE TO REMOVE THE 301 REDIRECTS FROM YOUR HTACCESS FILE OR THE PAGES WILL NEVER LOAD!!!!**

## Contributing

Please open an issue if you know better htaccess code snippets than are listed here, or if you want to add better desriptions and notes. I would like to make this readme file a go-to resource for developers.

<div align="right">&#8673; <a href="#back-to-top" title="Table of Contents">Back to Top</a></div>