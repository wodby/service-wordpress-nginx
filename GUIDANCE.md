# Nginx for WordPress on Wodby

What this service adds to the PHP Nginx service it is based on. Check it before adding rewrite rules or web server configuration for WordPress to the repository.

## Preset

The service sets `NGINX_VHOST_PRESET` to `wordpress`, the image's rule set for WordPress. Its template is declared as a config file of this service and can be overridden there. What it does:

- Paths that match no file or directory go to `index.php`, so permalinks need no configuration. `.htaccess` files are not read; rules that a plugin writes there have no effect.
- A `.php` path runs only when the file exists; a missing one returns 404.
- PHP files under an `uploads` or `files` directory are denied, as is `wp-content/uploads/woocommerce_uploads`.
- `readme.html`, files ending in `.txt`, `.pot`, `.sh` or `.sql`, `composer.json`, `composer.lock`, `package.json`, lock files, and plugins' `README.md`, `CHANGELOG.md` and similar return 404. Text files under `wp-content/uploads` are still served.
- `robots.txt` and `wp-sitemap*.xml` are passed to WordPress when no such file exists.
- Requests for `wp-admin` without a trailing slash are redirected, and the rewrites for a subdirectory multisite network are included.
- The default `Content-Security-Policy` allows framing by the same origin, and `Cache-Control` is set by Nginx.

## Settings and files shared with the PHP service

- The `docroot` setting (variable `DOCROOT_SUBDIR`) is taken from the linked PHP service. Change it there.
- The files volume of the linked PHP service is mounted here at `/mnt/files`.
- On start, the service's init action (`WODBY2_SERVICE_INIT_ACTION`, here `init`) makes `wp-content/uploads` under the WordPress root a link to `/mnt/files/public`, so uploads are served by Nginx directly from the volume. The link cannot be made when `wp-content/uploads` in the image holds anything other than a `.gitignore`.

Themes, plugins and WordPress core are served from the built image. A plugin or theme installed from the WordPress admin is written in the PHP service's container only, so its static files are not in this service; add plugins and themes through the repository.

## Variables specific to WordPress

- `NGINX_WP_FILE_PROXY_URL`: redirects requests for missing files under `wp-content/uploads` to another site, for an environment without a copy of the uploads.
- `NGINX_WP_GOOGLE_XML_SITEMAP`, `NGINX_WP_YOAST_XML_SITEMAP`: add the rewrites those sitemap plugins need.
- `NGINX_WP_NOT_FOUND_REGEX`: the pattern of paths that return 404.

## Check the result

- `nginx -T` shows the preset in effect.
- `ls -l <WordPress root>/wp-content/uploads` inside this service shows the link to the files volume.
