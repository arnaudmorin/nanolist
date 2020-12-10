nanolist
========

nanolist is a lightweight mailing list manager written in Go. It's easy to
deploy, and easy to manage. It was written as an antithesis of the experience
of setting up other mailing list software.

Usage
-----

nanolist is controlled by end users by emailing nanolist with a command in the subject.

The following public commands are available:

* `list`  - Reply with a list of available mailing lists
* `subscribe list-address`  - Subscribe to receive mail sent to the given list
* `unsubscribe list-address`  - Unsubscribe from receiving mail sent to the given list

nanolist can also be used from cli for admins:

```
$ nanolist --config /etc/nanolist/nanolist.ini --help
usage: nanolist [<flags>] <command> [<args> ...]

nanolist server

Flags:
  -h, --help       Show context-sensitive help (also try --help-long and --help-man).
      --debug      Don't send emails - print them to stdout instead
      --config=""  Load configuration from specified file

Commands:
  help [<command>...]
    Show help.

  check
    Check the configuration

  message
    Process a message from stdin

  list [<flags>]
    List all lists and their subscribers

  subscribe <address> <list>
    Subscribe to a list

  unsubscribe <address> [<list>]
    Unsubscribe from a list

  create [<flags>] <list>
    Create a list

  modify [<flags>] <list>
    Update a list

  delete <list>
    Delete a list

```

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

If you already clone code, local build:
``

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
database = /var/local/nanolist.db

[bot]
# Address nanolist should receive user commands on
command_address = lists@lists.example.com
bounces_address = bounces@lists.example.com

# SMTP details for sending mail
smtp_hostname = "smtp.lists.example.com"
smtp_port = 25
smtp_username = "nanolist"
smtp_password = "hunter2"
```

Initiate the DB and create a list by invoking nanolist:
```bash
# Regular list
nanolist create --name="RMCS" --description="Liste du Radio Model Club Senonais" rmcs@lists.example.com

# List with bcc (e.g. for archiving)
nanolist create --name="RMCS" --description="Liste du Radio Model Club Senonais" --bcc archive@lists.example.com rmcs@lists.example.com

# List restricted to some users (e.g. for announcements / newletters)
nanolist create --name="RMCS annonces" --description="Annonces du club" --poster admin@lists.example.com announce@lists.example.com

# Hidden list, for subscribers only
nanolist create --name "fight club" --flag subscribers_only --flag hidden fc@lists.example.com
```

Add users to the ml
```
nanolist subscribe arnaud@example.com rmcs@lists.example.com
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
# And make sure the SELECT statement for your list domain (lists.example.com) return 'nanolist'
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
