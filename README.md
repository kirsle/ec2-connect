# EC2 Connect Helper

This is a script to assist with managing SSH connections to multiple EC2
instances so that you don't have to log in to the Amazon console just to look
up that hostname that you want to connect to.

Conventional wisdom is to create shell scripts for this job, i.e.:

```bash
#!/bin/bash
ssh -i /path/to/keyfile.pem ec2-user@public-dns.amazonaws.com
```

This is okay if your EC2 instances stay the same for extended periods of time,
but if your instances are auto-scaling and occasionally rebuild themselves and
get new IP addresses and hostnames, this solution isn't very scalable.

Enter `ec2-connect`: it uses the Python `boto` API to look up your EC2 instances
by name and manages the SSH credentials by name (or ID) instead of hostname. So
you don't have to log in to the AWS Console to look up those hostnames when they
change.

Usage can be as simple as:

```bash
$ ec2-connect my-ec2-server
```

In case you have your auto-scale group set up to keep multiple instances of the
same name running at once (so that `my-ec2-server` matches two or more
instances), the script prompts you to choose one. You can also provide a `-n`
option to choose one at the command line to make it automatically answer that
question.

So now your "conventional wisdom" shell scripts can just call on this program
instead of including the hard-coded SSH command line.

# AWS Credentials

This script currently uses your global credentials found in your `~/.boto`
config file. The format of that file looks like this:

```ini
[Credentials]
aws_access_key_id = ACCESS KEY
aws_secret_access_key = SECRET KEY
```

You can probably also configure credentials with environment variables, but this
is untested.

# Configuration

On the first run, this script creates the config file
`~/.config/ec2-connect.ini` and exits. You then need to edit this file to fill
in SSH information for the various EC2 names (or IDs) that you control.

Example:

```ini
[my-ec2-server]
username = ubuntu
keyfile = /path/to/key.pem

[another-ec2-server]
username = ec2-user

[i-256da425]
username = ec2-user
keyfile = /path/to/another/key.pem
```

Create one section for each name or ID (you can specify to connect to an ID with
the `-i` command line option to the script), and provide the config fields
`username` and `keyfile` (if needed). These are the only supported options at
this time.

# Usage Examples

Connect to a single EC2 instance by name:

`ec2-connect my-ec2-server`

Connect to an EC2 instance by its ID instead of name:

`ec2-connect -i i-256da425`

Connect to an EC2 server that has multiple instances, and always choose the
second one (sorted by ID) rather than be prompted for which one:

`ec2-connect my-ec2-autoscale -n 2`

# Full Usage

```
usage: ec2-connect [-h] [--region REGION] [--id] [--number NUMBER] name

EC2 Connection Helper

positional arguments:
  name                  The name of the EC2 instance to connect to.

optional arguments:
  -h, --help            show this help message and exit
  --region REGION, -r REGION
                        AWS region to connect to. Default is us-east-1.
  --id, -i              Find instance by ID instead of name.
  --number NUMBER, -n NUMBER
                        If more than one instance matches the name, you will
                        be prompted to answer which one you want. Provide -n
                        to automatically choose that answer.
```

# Contributing

Pull requests are welcome. :smile: This script was just hacked together to fit
my needs, which were being able to SSH into EC2 instances using either LDAP
authentication or a key file, and to save me the time and effort of logging into
the Amazon console to look up that random odd hostname that changed its IP
address.

# License

This program is released under the MIT license. See `LICENSE` for more
information.

Copyright (C) 2015 Noah Petherbridge.
