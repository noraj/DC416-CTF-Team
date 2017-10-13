# The Robots Grandmother

## Description
Every once in a while we see the Grand Robot Leader Extraordinaire communicating over email with the Grand Robot Matriarch. We suspect there might be secret communications between the two, so we tapped into the network links at the Matriarch's house to see if we could grab the password to the account. We got this file, but our network admin is gone for two weeks training pigeons to carry packets. So we don't actually know how to read this file. Can you help us?

### Solution
We are given small SMTP session pcap file, with the initial EHLO and the smtp auth.

We can see that the authentication credentials are base64 encoded which we can easily decode.

```
~# tcpdump -r the-robots-grandmother.pcap 

18:11:46.835199 IP6 ip6-localhost.54050 > ip6-localhost.smtp: Flags [P.], seq 1:16, ack 63, win 342, options [nop,nop,TS val 221627326 ecr 221627039], length 15: SMTP: ehlo x.shh.sh
18:11:46.836051 IP6 ip6-localhost.smtp > ip6-localhost.54050: Flags [P.], seq 63:197, ack 16, win 342, options [nop,nop,TS val 221627327 ecr 221627326], length 134: SMTP: 250-x.shh.sh Hello x.shh.sh [::1]
18:11:46.836066 IP6 ip6-localhost.54050 > ip6-localhost.smtp: Flags [.], ack 197, win 350, options [nop,nop,TS val 221627327 ecr 221627327], length 0
18:11:49.043172 IP6 ip6-localhost.54050 > ip6-localhost.smtp: Flags [P.], seq 16:28, ack 197, win 350, options [nop,nop,TS val 221627547 ecr 221627327], length 12: SMTP: auth login
18:11:49.043346 IP6 ip6-localhost.smtp > ip6-localhost.54050: Flags [P.], seq 197:215, ack 28, win 342, options [nop,nop,TS val 221627547 ecr 221627547], length 18: SMTP: 334 VXNlcm5hbWU6
18:11:54.915343 IP6 ip6-localhost.54050 > ip6-localhost.smtp: Flags [P.], seq 28:42, ack 215, win 350, options [nop,nop,TS val 221628134 ecr 221627547], length 14: SMTP: bWFsbG9yeQ==
18:11:54.915496 IP6 ip6-localhost.smtp > ip6-localhost.54050: Flags [P.], seq 215:233, ack 42, win 342, options [nop,nop,TS val 221628134 ecr 221628134], length 18: SMTP: 334 UGFzc3dvcmQ6
18:12:00.154289 IP6 ip6-localhost.54050 > ip6-localhost.smtp: Flags [P.], seq 42:96, ack 233, win 350, options [nop,nop,TS val 221628658 ecr 221628134], length 54: SMTP: ZmxhZy1zcGluc3Rlci1iZW5lZml0LWZhbHNpZnktZ2FtYmlhbg==
```

When we decode the user base64 we get:

```
echo bWFsbG9yeQ==  | base64 -d
mallory
```

And password base64 is decoded to:

```
echo ZmxhZy1zcGluc3Rlci1iZW5lZml0LWZhbHNpZnktZ2FtYmlhbg | base64 -d
flag-spinster-benefit-falsify-gambianbase64
```

flag is: spinster-benefit-falsify-gambianbase64
