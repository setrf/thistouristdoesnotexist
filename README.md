thistouristdoesnotexist
=======================

A zero‑UI static site that returns one random image on every page load.

Live Domains
- https://thistouristdoesnotexist.com/
- https://www.thistouristdoesnotexist.com/

Behavior
- No HTML/JS UI; `GET /` responds with a single image file.
- Each refresh returns a random file from the `photos/` folder.
- Caching is disabled (no-store) so the image varies on every request.

Where To Put Photos
- Upload images via SFTP to: `/var/www/apps/thistouristdoesnotexist/photos/`
- Supported formats: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.svg`
- File permissions: typical `0644` (rw‑r‑‑r‑‑). The directory is owned by root and readable by nginx.
- You can add/remove files at any time; changes take effect immediately.

Nginx Configuration Summary
- Vhost: `/etc/nginx/sites-available/thistouristdoesnotexist.com` (enabled)
- Root rewrites to the photos directory and serves a random file; caching is disabled for randomness.

Snippet (what’s installed)
```
server {
  listen 443 ssl; listen [::]:443 ssl;
  server_name thistouristdoesnotexist.com www.thistouristdoesnotexist.com;

  # / -> /photos/ (internal)
  rewrite ^/$ /photos/ last;

  # Return a random file from photos (no caching)
  location = /photos/ {
    root /var/www/apps/thistouristdoesnotexist;
    random_index on;
    expires off;
    add_header Cache-Control "no-store, no-cache, must-revalidate, max-age=0" always;
  }

  # Direct file access (no listing; no caching)
  location ^~ /photos/ {
    root /var/www/apps/thistouristdoesnotexist;
    autoindex off;
    try_files $uri =404;
    expires off;
    add_header Cache-Control "no-store, no-cache, must-revalidate, max-age=0" always;
  }
}

server {
  listen 80; listen [::]:80;
  server_name thistouristdoesnotexist.com www.thistouristdoesnotexist.com;
  return 301 https://$host$request_uri;
}
```

HTTPS
- Enabled via Certbot for both apex and www (HTTP redirects to HTTPS).
- To renew/test: `sudo certbot renew --dry-run`

Troubleshooting
- Ensure there is at least one image in `photos/`.
- Verify direct access to a known file:
  - `curl -I http://thistouristdoesnotexist.com/photos/<filename>` → `200`
- Verify root returns an image (over HTTPS once DNS is set):
  - `curl -I https://thistouristdoesnotexist.com/` → `200` and `Content-Type: image/*`
- Logs: `/var/log/nginx/access.log`, `/var/log/nginx/error.log`

Notes
- Directory listing is disabled under `/photos/`.
- If you later want a minimal HTML wrapper instead, we can swap to a tiny static page that picks a random image client‑side.
