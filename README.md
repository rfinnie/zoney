# Zoney

Zoney is a program which takes YAML source files and builds BIND 9 zone files, optionally DNSSEC signing them in the process.

## Usage

Build any destination zones which are older than the source YAML file:
```shell
zoney-build
```

Build one or more individual zones:
```shell
zoney-build /etc/bind/zones/example.com.yaml /etc/bind/zones/example2.com.yaml
```

Explicitly build all zones, forcing the update of the serials (needed periodically for DNSSEC re-signing):
```shell
rndc freeze
zoney-build --update-serials /etc/bind/zones/*.yaml
rndc reload
rndc thaw
```

This system makes no attempt to re-integrate dynamically-updated zones at the BIND level.  The author's BIND server does allow for dynamic updates on a few zones, but they're just for Let's Encrypt wildcard verification and so changes do not need to be re-integrated.  The freeze/build/reload/thaw example above will safely discard any journal which has been created for a zone between builds.

If using DNSSEC signing, it is assumed the private keys (and by default, the public keys) are in the destination directory with the zones.

## Configuration

```yaml
# Zone name
# Required, no default
# If it does not end with a period, one will be added.
zone: example.com.

# Built zone filename format
# Default: '{zone}.zone'
# May be absolute, or relative to --dest-dir.
# {zone} will be replaced with the zone name (without ending period).
filename: 'example.com.zone'

# Whether to DNSSEC sign the zone
# Default: true if a key is detected, otherwise false
# A key is detected if there is an include (see below) which starts with
# "K" and ends with ".key"
sign: true

# Default record TTL
# Default: --ttl (3600)
ttl: 3600

# List of records to build
# Default: []
# Records may be built three different ways:
#   Full: dictionary of {name, ttl (optional), class (optional), type,
#     data}
#   Short: list of [name, type, data]
#   String: string of normal zone entries
# Name may be '@' for the zone name, a short host name, or a full host +
# zone name.
#
# Data is passed verbatim, so care must be taken to double string where
# appropriate. See the CAA and DKIM examples below.
#
# For SOA records, the serial will be replaced with an epoch time if the
# source serial is the placeholder "0" (highly recommended).
# The replacement serial will be the modification time of the YAML file,
# otherwise the current time if --update-serials.
records:
- ['@', SOA, ns1.example.com. dns.example.com. 0 3600 3600 2419200 3600]
- ['@', NS, ns1.example.com.]
- ['@', NS, ns1.example.com.]
- ['@', A, 192.0.2.1]
- ['@', AAAA, '2001:db8::1']
- ['@', CAA, '0 iodef "mailto:caa@example.com"']
- ['@', CAA, 0 issue "letsencrypt.org"]
- ['@', CAA, 0 issuewild "letsencrypt.org"]
- [www, A, 192.0.2.1]
- [www, AAAA, '2001:db8::1']
- name: mail
  type: A
  data: 192.0.2.2
- name: full.example.com.
  ttl: 60
  class: IN
  type: A
  data: 192.0.2.3
- [mail._domainkey, TXT, '"v=DKIM1; h=sha256; k=rsa; " "p=longbase64" "morebase64"']
- |
    raw1.example.com. IN A 192.0.2.4
    raw2.example.com. IN A 192.0.2.5

# Include files of normal zone entries
# Default: []
# May be absolute, or relative to --dest-dir.
include: [Kexample.com.+008+20974.key, Kexample.com.+008+48993.key]
```

## License

Copyright (C) 2022 [Ryan Finnie](https://www.finnie.org/)

This Source Code Form is subject to the terms of the Mozilla Public
License, v. 2.0. If a copy of the MPL was not distributed with this
file, You can obtain one at http://mozilla.org/MPL/2.0/.
