I was investigating a hacked website using the "alone" theme (https://www.wordfence.com/threat-intel/vulnerabilities/wordpress-themes/alone). There was recently a disclosed vulnerability in older versions of the theme:

https://thehackernews.com/2025/07/hackers-exploit-critical-wordpress.html

In the observed attacks, the flaw is averaged to upload a ZIP archive ("wp-classic-editor.zip" or "background-image-cropper.zip") containing a PHP-based backdoor to execute remote commands and upload additional files. Also delivered are fully-featured file managers and backdoors capable of creating rogue administrator accounts.
[..]
To mitigate any potential threats, WordPress site owners using the theme are advised to apply the latest updates, check for any suspicious admin users, and scan logs for the request "/wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin."

Initially, it looked like this was the POE, but I couldn't prove it as I couldn't match requests in the logs to /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin:

$ zcat ~/logs/2025-07/www.*.log.gz  | grep -v wp-cron | awk {'print $1,$4,$5,$6,$7,$9'} | grep alone_import_pack_install_plugin
185.159.158.108 [11/Jul/2025:22:22:21 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
185.159.158.108 [11/Jul/2025:22:22:22 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
185.159.158.108 [11/Jul/2025:22:22:23 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
62.133.47.18 [11/Jul/2025:22:24:08 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
62.133.47.18 [11/Jul/2025:22:24:08 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
62.133.47.18 [11/Jul/2025:22:24:09 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
146.70.10.25 [11/Jul/2025:22:26:53 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
146.70.10.25 [11/Jul/2025:22:26:54 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
146.70.10.25 [11/Jul/2025:22:26:54 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
74.118.126.111 [11/Jul/2025:22:30:55 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
74.118.126.111 [11/Jul/2025:22:30:55 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
74.118.126.111 [11/Jul/2025:22:30:55 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
188.215.235.94 [11/Jul/2025:22:33:46 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
188.215.235.94 [11/Jul/2025:22:33:47 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
188.215.235.94 [11/Jul/2025:22:33:48 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
188.215.235.94 [11/Jul/2025:22:35:31 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
188.215.235.94 [11/Jul/2025:22:35:32 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
188.215.235.94 [11/Jul/2025:22:35:32 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
85.203.32.13 [11/Jul/2025:22:36:05 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
85.203.32.13 [11/Jul/2025:22:36:05 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
85.203.32.13 [11/Jul/2025:22:36:05 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
188.215.235.254 [11/Jul/2025:22:39:11 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
188.215.235.254 [11/Jul/2025:22:39:12 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
188.215.235.254 [11/Jul/2025:22:39:12 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
198.145.157.102 [11/Jul/2025:22:42:59 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
198.145.157.102 [11/Jul/2025:22:43:00 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
198.145.157.102 [11/Jul/2025:22:43:00 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 400
165.22.49.136 [31/Jul/2025:19:51:28 -0400] "POST //wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 403
192.145.117.169 [31/Jul/2025:22:59:17 -0400] "POST /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 301
192.145.117.169 [31/Jul/2025:22:59:18 -0400] "GET /wp-admin/admin-ajax.php?action=alone_import_pack_install_plugin 403

Turns out, I was able to find a POST request directly to /wp-admin/admin-ajax.php:

87.120.92.24 - - [16/Jul/2025:15:47:15 -0400] "POST /wp-admin/admin-ajax.php HTTP/1.1" 200

and only a couple of seconds later:

87.120.92.24 - - [16/Jul/2025:15:47:30 -0400] "POST /wp-content/plugins/background-image-cropper/accesson.php

I couldn't examine the contents of the POST request, as they're no longer available, but it's safe to assume that this is the initial point of entry, given the fact that the oldest infected file is:

/wp-content/plugins/background-image-cropper/background-image-cropper.php 2025-07-16 21:27:58

So, I backed up the malicious installation, and then proceeded to reinstall it and secure it. I regularly use https://github.com/Dantata/cleaner when dealing with hacked accounts because it allows me to (1) run a pretty thorough pre-clean check, and then do a clean reinstall.

This particular case was interesting because (1) the attack was slightly modified in comparison with the public data. Even if it was just hiding the GET parameter and (2) because the bad actors tried to prevent restoration/cleanup by tightening the permissions of files/directories - some were set to 0555, others to 0444. This initially caused problems for cleaner.sh.
