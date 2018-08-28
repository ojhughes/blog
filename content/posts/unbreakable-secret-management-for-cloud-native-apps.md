---
title: "Unbreakable Secret Management for Cloud Native Applications"
date: 2018-08-16T10:12:36+01:00
draft: false
---
{{< figure src="http://i.imgur.com/iOVOxLR.jpg" title="This Message Will Self Destruct!" >}}

Secret management has a come a long way since Inspector Gadget's shenanigans however the idea of self destructing letters is still really relevant today. Discrete microservices enable enterprises to deliver new features at a much greater pace but this approach brings with it a much larger number of secrets and credentials that need to be protected. 

Leaking of credentials is a major cause of security breaches, luckily a number of tools and techniques have emerged for managing secrets. In this post we will explore how [Spring Boot](https://spring.io/projects/spring-boot), [CredHub](https://docs.cloudfoundry.org/credhub/) and [HashiCorp Vault](https://www.vaultproject.io) can work together to build an unbreakable microservices architecture and how they can be used to facilitate the [Three R's of Enterprise Security](https://builttoadapt.io/the-three-r-s-of-enterprise-security-rotate-repave-and-repair-f64f6d6ba29d)

## Feature comparison
[CredHub](https://docs.cloudfoundry.org/credhub/) and [HashiCorp Vault](https://www.vaultproject.io) have similar goals in managing credentials although CredHub has been built from the ground up to tightly integrate with Cloudfoundry. Below is a high level comparison of features as of August 2018;

Feature                         | CredHub | Vault |
| ------------------------------|:---------------:|---------------|
| Encrytped Credential Storage. | ✅ Pluggable, software based AES by default| ✅ Pluggable, software based AES by default |
| Hardware encryption support | ✅ Supported in Open Source | ⚠️ Enterprise version required |

## A Belt and Braces approach
{{< figure src="https://i.imgur.com/iUpB8WE.jpg" >}}