# zscrub

zscrub is a program to help ensure that ZFS pools are scrubbed periodically.
zscrub is intended to be run daily from cron or systemd.

For each ZFS pool listed (default is all pools), zscrub checks the last scrub
time of that pool.  If the pool has not been scrubbed, or if the scrub is more
than 'days' old (default 7 days), zscrub will start a scrub. zscrub will only
allow one scrub at a time to avoid running multiple scrubs in parallel.

If a pool has not been scrubbed after 'warn-days' days (default 14 days), a
warning is issued.

zscrub exits with return code 0 if all pools have been scrubbed within 'days'
days.

zscrub exits with 1 if at least one pool has an overdue scrub or some other
error occurred.

## Background

I switched to ZFS for my storage needs. However, automatic scrub does not seem
to be part of the ZFS on Linux project, nor included in my distribution (Arch).

I found the "zpool-scrub" project at https://github.com/stesind/zpool-scrub.
This gave me the idea to use systemd to manage scrubs. However, I found that
the zpool-scrub project did not meet all of my needs and it seemed that the
project is in maintenance mode only. Thus rather than contributing to that
project, I started a new project. None of the code is the same anyway.

## Configuration

By default, the service will scrub all ZFS pools on the system once every 7
days and give a warning if any given pool has not been scrubbed successfully
after 14 days.

To change this behavior, edit the zscrub.service file and change the following line:

    ExecStart=/usr/bin/scrub-check

To something like this (which shows all options):

    ExecStart=/usr/bin/scrub-check pool1 pool2 --days 5 --warn-days 7 --verbose

## Installation

    sudo install -m 755 -o root -g root zscrub /usr/bin
    sudo install -m 644 -o root -g root zscrub.service /etc/systemd/system
    sudo install -m 644 -o root -g root zscrub.timer /etc/systemd/system
    
    sudo systemctl daemon-reload
    sudo systemctl enable --now zscrub.timer
