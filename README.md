

Installation & Administrator Guide · Version 1.3.2
IronVeil Security

Layered protection for WordPress, by Jassim T Mohammad. This guide covers installing the plugin, setting it up and using every feature.

    Firewall
    Login protection
    Admin identity verification
    Malware scanner & bulk clean
    Quarantine
    Server security scan
    Signed activity log

Contents

    Requirements
    Install and activate
    First-time setup checklist
    Administrator identity verification
    Firewall
    Malware scanner, bulk clean and quarantine
    Server scan
    Login security
    Alerts
    Reporting a bug
    Emergency recovery
    WP-CLI commands
    Uninstalling

1. Requirements
Item	Minimum
WordPress	6.2
PHP	7.4 (8.2 or newer recommended)
PHP extensions	sodium (bundled with PHP 7.2+, or WordPress’s sodium_compat), openssl, json
Optional	ClamAV daemon (clamd), WP-CLI, a persistent object cache for the global rate limit
2. Install and activate

    In WordPress, go to Plugins → Add New → Upload Plugin.
    Choose ironveil-security-1.3.2.zip and click Install Now.
    Click Activate.

Installing by FTP instead: unzip the file, upload the ironveil-security folder to wp-content/plugins/ with FTP or your host’s File Manager, then activate it under Plugins.
Updating from 1.2.x

Your settings are kept, and the new firewall rule groups and alert types switch on automatically. Since version 1.3.1, two-factor authentication is no longer included: updating deletes stored 2FA secrets and recovery codes, and administrators verify sensitive actions with emailed codes instead.
3. First-time setup checklist

Work through these steps once, in this order.

    Check your email address. Identity verification sends codes to each administrator’s account email, so make sure it’s correct under Users → Profile and that your site can send email (an SMTP plugin helps).
    Check your IP detection. Open IronVeil → Firewall. If your site is behind Cloudflare or a load balancer, set Visitor IP source and Trusted proxies (for example cloudflare). Without this, the firewall sees the proxy’s address instead of the visitor’s.
    Run a malware scan. Open IronVeil → Scanner, click Scan now and review the findings (section 6).
    Run a server scan. Open IronVeil → Server Scan and click Scan the server now (section 7).
    Review the dashboard. IronVeil → Dashboard shows your security score and a checklist of what to fix.
    Optional: enable threat feeds. On IronVeil → Firewall, tick Spamhaus DROP under Threat intelligence IP feeds and save.
    Optional: hide the login page. On IronVeil → Login Security, set a Custom login URL slug and bookmark the new address.
    Check alerts. On IronVeil → Alerts & Tools, confirm the alert email address.

4. Administrator identity verification

One verification protects everything sensitive on an administrator account. You verify once, and IronVeil trusts your current browser session for a short window (15 minutes by default).
What needs verification

    All IronVeil screens and actions, except Report a Bug.
    Installing, uploading, editing, activating, deactivating and deleting plugins and themes.
    Adding or deleting users and changing roles.
    Saving any profile, including changing an email address or password.
    Saving any site settings (everything that posts to options.php).
    Exporting content, and exporting or erasing personal data.
    Creating or deleting application passwords, and REST changes to plugins, themes, settings and users.

Editors and other non-administrators are not affected.
How to verify

When you open a protected screen, IronVeil shows a Verify it’s you page. Click Email me a code and enter the 6-digit code sent to your account email. The code expires after 10 minutes and works only in the browser that requested it.

While you’re verified, IronVeil screens show a green Verified label and a Lock now link that ends the window immediately.
Built-in protections

    After 5 wrong codes, code checks for that account pause for 15 minutes. The event is logged as critical and the site owner is emailed.
    At most 6 emailed codes are sent per hour.
    The verified window is stored inside the WordPress session. It ends at logout and doesn’t carry over to a cookie stolen from another session.
    A form submitted while not verified isn’t saved. Verify, then submit it again; a notice reminds you.

Settings (IronVeil → Login Security)
Setting	Default	Notes
Administrator identity verification	On	Turn off only if you accept the risk.
Stay verified for (minutes)	15	Between 5 and 240.

Remote management tools such as MainWP or ManageWP that use application passwords keep working. Only a verified administrator can create an application password in the first place.
5. Firewall
Setting	What it does
Firewall mode	Protect blocks attacks. Monitor only logs them, which is useful for a few days after setup to check for false positives. Disabled turns the firewall off.
Rule groups	SQL injection, XSS, path traversal, remote code execution, object injection, malicious uploads, vulnerability probes, attack tools, SSRF, XXE, CRLF injection and HTTP protocol enforcement.
Threat intelligence feeds	Spamhaus DROP (criminal networks) and Tor exit nodes, downloaded daily. Tor blocking also blocks privacy-conscious visitors.
Auto-block	Blocks an IP after repeated attacks, 404 floods or repeated login lockouts.
Allowlist	IPs and CIDR ranges that are never blocked. Add your office or home IP here.
Parameter allowlist	Request field names the firewall skips, for plugins that legitimately send code.
Extended protection (Pro)	Loads the firewall before every other plugin.
A legitimate request was blocked

The visitor sees a reference code. Search for it in IronVeil → Activity Log to see which rule matched, then add the field name to the Parameter allowlist or the visitor’s IP to the Allowlist.
You locked yourself out

Add define( 'IRONVEIL_DISABLE_FIREWALL', true ); to wp-config.php, log in, remove your IP from the blocklist, then delete the line.
6. Malware scanner, bulk clean and quarantine

Open IronVeil → Scanner. Scan now checks new and changed files; Deep scan re-checks every file.

Findings are grouped into Open, Ignored, Fixed and Resolved. Each finding shows its severity, what was found, where it is, and what Clean will do.
Acting on many findings at once

    Tick the checkboxes next to the findings, or use Select: All / Critical / High+ / Medium+ / None, or the box in the table header.
    Choose an action: Clean (apply the recommended fix), Quarantine files or Ignore.
    Optional: tick If a file cannot be cleaned safely, quarantine it instead.
    Click Apply to selected and confirm.

The result lists what was done, what failed, and what needs manual review. Each row also has its own Clean, Quarantine and Ignore buttons.
What Clean does
Finding	Clean does this
Modified WordPress core file	Replaces it with the official copy for your WordPress version, verified against the wordpress.org checksum.
Modified file of a wordpress.org plugin	Replaces it with the official copy for the installed version, verified against the wordpress.org checksum.
Web shell, PHP in uploads, unknown file in the WordPress root or core folders	Moves the file to the quarantine.
Injected code in your theme or a custom plugin	Removes only the injected block or statement. The result is syntax-checked and re-scanned before it’s written, and the original is kept in the quarantine.
Injected script in a post or widget	Removes the malicious <script> or hidden <iframe>. A post revision is saved first.
Outdated or vulnerable plugin or theme	Updates it to the latest version.
Server finding	Applies the server fix (section 7).
Closed or abandoned plugin, changed-file notice	Shown as needs manual review, with advice.

Safe by design: if almost a whole file is malicious, or removing the code would break the file, Clean refuses and changes nothing. Use Quarantine, or tick the fallback option, for those files.
Quarantine

Quarantined files are stored inert (base64-encoded) in wp-content/ironveil-quarantine, which is protected from web access. Tick items to Restore them to their original location or Delete permanently. Items marked backup are copies taken before IronVeil changed a file; to restore one, first remove or rename the current file at that location.
Core files

For modified core files, View changes shows the difference from the official file, and Download original gives you the official file to upload yourself.
7. Server scan

Open IronVeil → Server Scan and click Scan the server now. By default it also runs weekly.
Area	Checks
PHP	End-of-life version, allow_url_include, display_errors, expose_php, shell functions, open_basedir, unexpected auto_prepend_file, PHP running as root, free disk space
Database	End-of-life MySQL or MariaDB, server-wide privileges, empty or short password
Files	wp-config.php permissions, world-writable code folders, wp-config backups, .env files, database dumps, backup archives, public logs, phpinfo/Adminer/installer leftovers, .git folders, PHP files in /tmp or /dev/shm, must-use plugins to review
Accounts	Administrators created in the last 7 days, number of administrators, WP_DEBUG
HTTP	Security headers, version disclosure, HTTP-to-HTTPS redirect, TLS certificate expiry, directory listing, downloadable debug.log, .git or .env, TRACE method
Network	MySQL, PostgreSQL, Redis, Memcached, MongoDB, Elasticsearch, FTP and Telnet listening on the server’s public address

Problems appear as findings with checkboxes, handled just like malware findings. Clean fixes these automatically:

    Sets wp-config.php permissions to 640.
    Removes world-write permission from the listed paths.
    Quarantines exposed files (up to 50 MB each).
    On Apache or LiteSpeed, writes protective rules to the root .htaccess that block directory listing and downloads of logs, backups, .env and .git.

Everything else, such as PHP settings, database privileges and open ports, shows the exact change to make. Ask your host to apply it.

Port results: the check runs from the server itself. A port reported as listening may still be blocked from the internet by your firewall, so confirm with your host.

nginx: nginx ignores .htaccess. Add the rules shown on IronVeil → Hardening to your server block.
8. Login security

    Lockouts. After 5 failed logins in 15 minutes, an IP is locked out for 30 minutes. After 3 lockouts in 24 hours, it’s blocked for a day.
    Targeted accounts. When one username receives many failed logins in an hour, only IPs that have logged in to it successfully before are accepted.
    Breached passwords. New passwords found in known data breaches are rejected, and privileged users with a breached password are flagged on the Login Security page.

9. Alerts

On IronVeil → Alerts & Tools, choose which events email you. Alerts are throttled so an attack can’t flood your inbox. The Free edition always includes new administrator and verification lockout alerts.
10. Reporting a bug

Open IronVeil → Report a Bug.

    Write a short summary and describe what happened. Add steps to reproduce if you can.
    Check the reply-to address.
    Read the Diagnostic information panel. That exact text is added to the report if the box is ticked. Secrets such as the license key, API tokens, email addresses, IP lists, the login URL and server paths are removed.
    Tick the consent box and click Send bug report.

Reports go to ironveil.wpplug@gmail.com through your site’s email setup. Nothing is ever sent automatically. If your site can’t send email, the page shows an Open in my email app button with the report filled in. You can send at most 3 reports per hour.

If IronVeil records an internal error, its screens show a notice with a link to this page.
11. Emergency recovery

Add one of these lines to wp-config.php, above /* That's all, stop editing! */. Remove it once the problem is fixed.
Use it when	Line to add
You blocked yourself	define( 'IRONVEIL_DISABLE_FIREWALL', true );
You forgot the custom login URL	define( 'IRONVEIL_DISABLE_LOGIN_SLUG', true );
You can’t verify because your site can’t send email	define( 'IRONVEIL_DISABLE_VERIFY', true );
You want to delete the plugin but keep its data	define( 'IRONVEIL_KEEP_DATA', true );
12. WP-CLI commands

wp ironveil status                          # security score and checklist
wp ironveil scan [--deep]                   # full malware scan, lists finding ids
wp ironveil clean 12 15                     # clean findings by id
wp ironveil clean --min-severity=4 --fallback
wp ironveil clean 20 --action=quarantine    # or --action=ignore
wp ironveil server-scan                     # server security scan
wp ironveil update-feeds                    # refresh threat-intelligence feeds
wp ironveil block 203.0.113.7 --hours=24
wp ironveil unblock <ip|all>
wp ironveil unlock-verify <user>            # clear a verification lockout
wp ironveil firewall <block|monitor|off>
wp ironveil reset-login-url
wp ironveil clamav-test
wp ironveil update-signatures
wp ironveil license <status|activate|deactivate> [<key>]

13. Uninstalling

    Deactivating removes scheduled tasks, the early-loader file and IronVeil’s .htaccess blocks, but keeps your data.
    Deleting the plugin from Plugins also removes its database tables, settings, user data and the quarantine folder, unless IRONVEIL_KEEP_DATA is defined.
