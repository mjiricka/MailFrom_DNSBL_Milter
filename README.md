# Milter for DNSBL check of MAIL FROM address

This Python milter (milter = mail filter) takes domain from MAIL FROM address,
gets the first A record for this domain and checks whether this address is
in [ZEN Spamhaus.org](https://www.spamhaus.org/zen/) database. I empirically
found out that such restriction would filter all spam that came through my Postfix
mail server.

[Postfix](http://www.postfix.org/) is a great mail server but it lacks something
like `reject_rbl_client` restriction, but for a sender. It has `reject_rhsbl_sender`
but it seems spammers have more domains then IP addresses (which is not surprising?)
So there is a lot of domains that are not in DNSBL databases but point to an IP that
is in the database.

Hopefully Postfix will support this type of check one day, for more info see
[this thread](http://postfix.1071664.n5.nabble.com/Why-there-is-no-reject-rbl-sender-restriction-td91739.html).
Until then one can use this milter.


## Requirements

Python 2 is needed and two python modules: [pymilter](https://pythonhosted.org/pymilter/) and
[dnspython](http://www.dnspython.org/). Both in PIP:

    pip install pymilter
    pip install dnspython

Module `pymilter` has further dependencies, e.g. `libmilter` is needed with its development files
(usually in an extra package like `libmilter-dev` or packed with sendmail development files).

Usually is also needed own DNS server, see [ZEN Spamhaus.org](https://www.spamhaus.org/zen/)
for more details.


## Installation and usage

When requirements are met the milter itself can be installed. This is an example for Postfix
running in chroot:

    # Prepare directory for milter's socket:
    mkdir /var/spool/postfix/var/run/dnsblmilter/
    chown postfix /var/spool/postfix/var/run/dnsblmilter/
    
    # Copy milter and its systemd file:
    cp dnsblmilter /usr/local/bin/
    cp dnsblmilter.service /etc/systemd/system/dnsblmilter.service

Add this line into `/etc/postfix/main.cf`:

    smtpd_milters = unix:/var/run/dnsblmilter/socket

And finally start it:
    
    systemctl start dnsblmilter
    systemctl restart postfix

BTW, do not forget to enable it later:

    systemctl enable dnsblmilter


## Possible improvements

 - [ ] Make script more like a daemon so it could be at least stopped.
 - [ ] Instead of hard coded Spamhaus service add support for more customizable services?

