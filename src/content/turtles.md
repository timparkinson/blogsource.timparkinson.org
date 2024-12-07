+++
date = '2024-12-07T05:12:47Z'
draft = false
title = 'Turtles'
description = "all the way down"
tags = ["tech", "spiffe", "identity"]
+++

I've been re-reading [Solving the Bottom Turtle](https://spiffe.io/book/). This describes SPIFFE, the Secure Production Framework for Everyone and it's reference implementation SPIRE, the SPIFFE Runtime Environment. This describes a possible solution to the problems of identity and secrets management in modern technology stacks. The [Turtles all the way down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down) expression refers to the proliferation of secrets in computing systems, where to protect any set of secrets you end up needing another secret to unlock them. And then a secret to protect that one, and so on, either infinitely or to the point that there is an unprotected secret. The problem is also known as "secret zero"

It's a problem that I've mused about a number of times in the work environment as well as for personal projects. There'll be times when I'm developing a service or scheduled task to perform some operation and that needs to access some other system such as a web service or a database. That other system will invariably need to be authenticated and authorised via a secret of some sort, such as an API key or a traditional username and password combo. Because a lot of the development I do is Windows and Active Directory based I can detour the problem using {{< mark >}}Kerberos{{< /mark >}} and things like [Group Managed Service Accounts](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-managed-service-accounts/group-managed-service-accounts/group-managed-service-accounts-overview). But that's typically only useful when both the consuming and providing systems are Windows adjacent [^adnote].

The more we move into the world of clouds, containers, microservices and CI/CD the more the problem is compounded. The danger is shared secrets that are hard to deliver non-disruptively and therefore hard to revoke and renew. 

SPIFFE aims to tackle these problems by creating a "dial tone" for authentication. The solution is based on short expiry, automatically updating, {{< mark >}}X.509{{< /mark >}} certificates in a chain of trust. The platform, which could be a cloud or Kubernetes cluster or something else, is capable of attesting that a workload has a certain identity. The providing service, which trusts the platform to attest it's workloads, and the consuming workload talk to each other using [mTLS](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/) [^mtlsisawesome]. There's a great deal of flexibility in how one constructs the identifiers and even the trust domains, so it's something that can fit the shape of different use cases. 

With my developer hat on [^notadeveloper] I like that it's reducing complexity, reducing the need to even think about certain concerns. With my infrastructure hat on I recognise that there are a lot of considerations in how one might implement this framework, but it definitely has a lot of promise to solve some very real problems. 

I'm not going to rehash the whole almost 200 pages of the [book](https://spiffe.io/book/). It's definitely worth a read, even if this is not the kind of thing that most people find themselves thinking about at 05:00 when everyone else is asleep. Maybe I am an odd fish.

[^adnote]: I've got a large amount of code that does filesystem-level actions on Enterprise NAS systems, provisioning, deprovisioning, etc. These are naturally AD integrated. 

[^mtlsisawesome]: mTLS is awesome: https://www.youtube.com/watch?v=FDAUwMwVuns&ab_channel=SecurityFest

[^notadeveloper]: My role at work is a sysadmin, so I'm not a developer per se. I do, however, find myself writing quite a lot of code. As a developer I don't want to care about 'plumbing', as a sysadmin I **am** a plumber.