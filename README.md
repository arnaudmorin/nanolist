nanolist
========

nanolist is a lightweight mailing list manager written in Go. It's easy to
deploy, and easy to manage. It was written as an antithesis of the experience
of setting up other mailing list software.

Usage
-----

nanolist is controlled by emailing nanolist with a command in the subject.

The following commands are available:

* `help` - Reply with a list of valid commands
* `list` - Reply with a list of available mailing lists
* `subscribe list-id` - Subscribe to receive mail sent to the given list
* `unsubscribe list-id` - Unsubscribe from receiving mail sent to the given list

Frequently Asked Questions
--------------------------

### Is there a web interface?

No. If you'd like an online browsable archive of emails, I recommend looking
into tools such as hypermail, which generate HTML archives from a list of
emails.

If you'd like to advertise the lists on your website, it's recommended to do
that manually, in whatever way looks best. Subscribe buttons can be achieved
with a `mailto:` link.

### How do I integrate this with my preferred mail transfer agent?

I'm only familiar with postfix, for which there are instructions below. The
gist of it is: have your mail server pipe emails for any mailing list addresses
to `nanolist message`. nanolist will handle any messages sent to it this way,
and reply using the configured SMTP server.

### Why would anyone want this?

Some people prefer mailing lists for patch submission and review, some people
want to play mailing-list based games such as nomic, and some people are just
nostalgic and/or crazy.

Installation
------------

First, you'll need to build and install the nanolist binary:
`go get github.com/arnaudmorin/nanolist`

Second, you'll need to write a config to either `/etc/nanolist.ini`
or `/usr/local/etc/nanolist.ini` as follows:

You can also specify a custom config file location by invoking nanolist
with the `--config` flag: `nanolist --config=/path/to/config.ini`

Example of configuration file:
```ini
# File for event and error logging. nanolist does not rotate its logs
# automatically.
# You'll need to set permissions on it depending on which account your MTA
# runs nanolist as.
log = /var/log/nanolist.log

# An sqlite3 database is used for storing the email addresses subscribed to
# each mailing list.
# You'll need to set permissions on it depending on which account your MTA
# runs nanolist as.
database = /var/db/nanolist.db

[bot]
# Address nanolist should receive user commands on
command_address = lists@example.com

# SMTP details for sending mail
smtp_hostname = "smtp.example.com"
smtp_port = 25
smtp_username = "nanolist"
smtp_password = "hunter2"
```

Initiate the DB and create a list by invoking nanolist:
```bash
nanolist create --list=golang@example.com --name="Go programming" --description="General discussion of Go programming" --bcc archive@example.com --bcc datahoarder@example.com
nanolist create --list=announce@example.com --name="Announcements" --description="Important announcements" --poster admin@example.com --poster moderator@example.com
nanolist create --list=robertpaulson99@example.com --name "fight club" --flag subscribers_only --flag hidden
```

More in help"
```bash
nanolist --help
```

Postfix configuration
---------------------

In `main.cf`:

```
# If nanolist is alone on this mail server
virtual_transport = nanolist

# Or if in combination with dovecot
virtual_transport = virtual
transport_maps = mysql:/etc/postfix/mysql-virtual-transports.cf
# And make sure the SELECT statement for your list domain (example.com) return 'nanolist'
```

There is a way to do that with hash also instead of mysql. Check postfix manual.

In `master.cf`, at the end of the file:
```
nanolist  unix  -       n       n       -       -       pipe
  flags=FR user=vmail argv=/path/to/nanolist
  message
```

and restart postfix.

Congratulations, you've now set up mailing lists!

License
-------

nanolist is made available under the BSD-3-Clause license.
