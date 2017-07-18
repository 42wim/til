# Conntrack updates

If you have a redundant firewall setup, and do a failover with conntrackd (which syncs the state between the two firewalls).
BUT forgot to enable UDP in the syncing, you can get to a state where a previously established (ASSURED) udp connection becomes UNREPLIED.

The result of this is that every packet of the flow fails to pass ESTABLISHED state in your firewall rules because according to conntrack it isn't established.  
And maybe it doesn't come established every again because the destination isn't required to respond to UDP packets.
Which causes every packet to pass the rules and to get logged. (with a videostream we could be speaking about 1000pps for a single stream).

So such a connection looks like this:

```
# conntrack -L |grep udp | grep UNREPLIED
udp      17 29 src=10.83.7.106 dst=10.131.97.13 sport=50004 dport=42880 [UNREPLIED] src=10.131.97.13 dst=10.83.7.106 sport=42880 dport=50004 mark=0 use=1
```

To fix this, we can update the state manually.

```
# conntrack -U --src 10.83.7.106 --dst 10.131.97.13 -u SEEN_REPLY
udp      17 29 src=10.83.7.106 dst=10.131.97.13 sport=50004 dport=42880 src=10.131.97.13 dst=10.83.7.106 sport=42880 dport=50004 mark=0 use=2
```

And ofcourse don't forget to enable UDP syncing ;-)
