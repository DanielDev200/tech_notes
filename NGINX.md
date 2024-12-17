# NGINIX

Messing with Nginix was DIFFICULT. TLDR is that web apps cannot be served from /root. Instead use /var/www and create that path if it doesn't exist.

Here is the summary of what went wrong:

### What the Issue Was

The root cause of the issue was file permission problems on the frontend/build directory. Nginx was unable to serve the index.html file because the permissions did not allow the Nginx user (www-data) to read the files. Despite updating permissions with chmod and chown, the issue persisted because the parent directories leading to the build folder (/root and /root/news_agg/frontend) also lacked appropriate permissions.

This was evident in the repeated "permission denied" errors in the Nginx logs when attempting to serve files from the build directory.

### What We Mistakenly Thought the Issue Was
Nginx configuration: We initially suspected that Nginx might not be properly configured to point to the correct frontend/build directory. This led us to:

1. Verify the nginx.conf and news_agg site configuration.
2. Ensure try_files and cache-control headers were correctly set.

### Outdated index.html or caching issues:
We considered that the index.html file in the build directory might be outdated or that Nginx was serving a cached version of the file. This led to:

1. Running npm run build to regenerate the production build.
2. Clearing Nginx's cache with sudo rm -rf /var/cache/nginx/*.
3. Using add_header Cache-Control to disable caching.

### Browser caching issues
We tested the app in incognito mode and used cache-busting techniques like appending query strings to URLs.

### Nginx not reloading configuration properly
We thought sudo systemctl reload nginx might not be picking up changes, leading to repeated configuration tests and full restarts of Nginx.

### Wrong file being served by Nginx
We explored whether Nginx was serving an outdated or incorrect file by using curl to verify the contents served by Nginx.

## What It Actually Ended Up Being
The permissions on the /root directory were too restrictive. Nginx (running as the www-data user) could not access the frontend/build directory because it couldn't traverse the /root directory.
Fixing this required granting execute (+x) permissions to /root and /root/news_agg/frontend.

## Learnings for the Future

### Check Permissions Early:

When you see permission denied errors in logs, immediately check the full directory path leading to the affected file, not just the file itself.

### Log Analysis is Key:

Regularly consult the error logs (/var/log/nginx/error.log) for clues. They often pinpoint the issue more precisely than trial-and-error debugging.

### Verify with curl:

curl is a great way to verify what your server is serving. Pairing it with tail on the error log can help pinpoint issues in real time.

### Understand Nginx Context:

Nginx requires read and execute permissions (+r and +x) for all directories in the path it serves files from. Permissions issues aren't always about the file itself.

### Default Nginx Behavior:
If you're using /root for serving files (which is unconventional), understand that it's a sensitive directory with restrictive permissions by default. Consider using a directory like /var/www/ or /srv/ for serving public files.

### Process of Elimination:
Debugging complex issues is a process of elimination. Systematically rule out:

1. Configuration issues
2. File freshness issues (rebuilds)
3. Caching (Nginx or browser)
4. Permissions issues

## Future Best Practices

### Choose the Right Directory:
Avoid serving files from /root. Use /var/www or /srv, which are designed for web applications and already have proper permissions for services like Nginx.

### Automate Deployment
Use tools like CI/CD pipelines or deployment scripts to ensure consistent builds, permissions, and configurations.

### Documentation
Document steps and fixes for future reference. You’ve already learned a lot here that can save you time in the future.

### Run Preflight Checks:
Before deploying:

1. Test the app locally.
2. Verify Nginx configurations with nginx -T.
3. Check file permissions for all directories in the path.

By applying these learnings, you can avoid similar pitfalls in the future and troubleshoot issues more efficiently. 
