---
title: "Migrate DNS From Squarespace to AWS Route53"
date: 2020-05-09T16:13:47+08:00
draft: false
tags: ["route53","squarespace", "dns", "aws"]
---

If you have a domain with Squarespace and would like to transfer it to Route53, the following steps will show you how.

It took me quite sometime to do it, and I hope this write up saves your time.

Steps to follow :

- Create a Route53 hosted zone
- Take note of the created NS and SOA (don't edit) in Route53
- Copy the four namservers from Route 53 and paste them in the Squarespace advanced DNS
- 'Create Record Set' in Route53. Alias -> no. Copy and paste the Ipv4 addresses from Squarespace (Website defaults)
- Decrease TTL of NS record in the hosted zone 1m - 15m in AWS Route53
- Copy and paste the MX Records from Squarespace to new record set with 'MX' type for email configuration
- Wait 2 days for TTL to expire. After 2 days, increase the TTL ()
- Monitor traffic, if unusual revert back to squarespace NS

References :

- [AWS : Migrate DNS Domain In Use](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html)
- [Squarespace : Advanced DNS Settings](https://support.squarespace.com/hc/en-us/articles/205812348-Advanced-DNS-settings)

[@omar16100](https://twitter.com/omar161000)
