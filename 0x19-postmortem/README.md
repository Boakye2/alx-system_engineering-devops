# 0x19. Postmortem


# Post mortem

Upon release of the Holberton School System Engineering and DevOps Project 0x17,
at approximately 12:00 a.m. Pacific Standard Time (PST), an outage occurred on a
Ubuntu 14.04 container running Apache web server. GET requests to the server led to
`500 Internal Server Error`, when the expected response was an HTML file defining a
simple WordPress site.

## Debug process

Bug Debugger Brennan (BDB...as in my initials...made this up on the spot, pretty
well, huh?) encountered the problem when opening the project and was instructed to
address, around 7:20 p.m. PST. He quickly set about solving the problem.

1. Checking running processes using `ps aux`. Two `apache2` processes - `root` and `www-data` -
were working properly.

2. Looked in the `sites-available` folder in the `/etc/apache2/` directory. determined that
the web server was serving content located in `/var/www/html/`.

3. In a terminal, run `strace` on the Apache process PID `root`. In another, rolled up
the server. I expected great things...only to be disappointed. `strace` gave no use
information.

4. Repeat step 3 except on process PID "www-data". I kept the expectations lower
time... but was rewarded! `strace` reveals error `-1 ENOENT (No such file or directory)`
occurring when trying to access the file `/var/www/html/wp-includes/class-wp-locale.phpp`.

5. I went through the files in the `/var/www/html/` directory one by one, using the Vim template
matching to try to locate the wrong `.phpp` file extension. Located in the
`wp-settings.php` file. (Line 137, `require_once(ABSPATH.WPINC.'/class-wp-locale.php');`).

6. Removed the final "p" from the line.

7. Tested another `curl` on the server. 200 A-ok!

8. Write a Puppet manifesto to automate error correction.

## Summation

In short, a typo. I must love them. In total, the WordPress application was experiencing a critical issue
error in `wp-settings.php` when trying to load `class-wp-locale.phpp` file. The maid
filename, located in the `wp-content` directory of the application folder, was
`class-wp-locale.php`.

The fix involved a simple correction of the typo, removing the trailing "p".

## Prevention

This failure was not a web server error, but an application error. To avoid such failures
moving forward, please keep the following in mind.

* Test! Trial trial trial. Test the application before deploying it. This error would have occurred
and could have been dealt with earlier if the application had been tested.

* Status monitoring. Enable certain uptime monitoring services such as
[UptimeRobot](./https://uptimerobot.com/) to instantly alert in case of website downtime.

Note that in response to this error I wrote a Puppet manifesto
[0-strace_is_your_friend.pp](https://github.com/Boakye2/alx-system_engineering-devops/blob/master/0x17-web_stack_debugging_3/0-strace_is_your_friend.pp)
to automate the correction of these identical errors should they occur in the future. Manifest
replaces all `phpp` extensions in the `/var/www/html/wp-settings.php` file with `php`.


