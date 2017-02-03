# Dovecot and envelope recipient address for sieve filtering
I'm using a [fork](https://github.com/42wim/docker-mailserver) of [docker-mailserver](https://github.com/tomav/docker-mailserver)

## The problem
When using dovecot lmtp the envelope recipient address (x-original-to) isn't passed to dovecot lmtp.
This means we can't filter the original recipients correctly with sieve.
It only sees the final expanded address.

For example, I've got user1@mydomain.tld, user2@mydomain.tld and user3@mydomain.tld which expand to user@myotherdomain.tld
```
Trying 1.2.3.4...
Connected to mymailserver.tld.
Escape character is '^]'.
220 mymailserver.tld ESMTP Postfix (Ubuntu)
helo mysender.tld
250 mymailserver.tld
mail from: joske@randomdomain.tld
250 2.1.0 Ok
rcpt to: user1@mydomain.tld
250 2.1.5 Ok
rcpt to: user2@mydomain.tld
250 2.1.5 Ok
rcpt to: user3@mydomain.tld
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
abc
.
250 2.0.0 Ok: queued as 764A2160A5BC
quit
221 2.0.0 Bye
```

This will end up with Delivered-To without the original recipients.
```
Return-Path: <joske@randomdomain.tld>
Delivered-To: user@myotherdomain.tld
```
So now way to filter on recipient anymore. I want to filter mail to user2@mydomain.tld into a specific mailbox

## First failed solution
Using PREPEND and smtpd_recipient_restrictions as found on <http://postfix.1071664.n5.nabble.com/virtual-alias-maps-and-X-Original-To-tp9124p9127.html>
works with 1 recipient, but if multiple recipients are specified the X-original-to for each recipient will be added.
Which gives issues for filtering.

## Working solution
Using pipe backend with dovecot-lda. By using the pipe-backend x-original-to can be added using a flag.

As I'm using docker-mailserver, this means reverting this patch <https://github.com/tomav/docker-mailserver/pull/360/files>
 and adding ```-O``` flag to dovecot in master.cf. (See <http://www.postfix.org/pipe.8.html> for flags)

main.cf
```
virtual_transport = dovecot # use dovecot instead of lmtp
dovecot_destination_recipient_limit = 1 # single recipient necessary (see pipe.8.html)
```

master.cf
```
dovecot   unix  -       n       n       -       -      pipe
  flags=DROhu user=docker argv=/usr/lib/dovecot/deliver -f ${sender} -a ${original_recipient} -d ${user}@${nexthop} -m ${extension}
```

The issue with this solution is that's less performant than using lmtp (spawning a process vs long-living process)

Some more information about this issue: <http://postfix.1071664.n5.nabble.com/pipe-flags-vs-lmtp-td11587.html>
