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
| :------------------------------|:---------------|:---------------|
| Encrytped Credential Storage (at rest). | ✅ Pluggable, software based AES by default| ✅ Pluggable, software based AES by default |
| Encrytped transport (in flight). | ✅ TLS| ✅ TLS |
| Pluggable secret storage backends | ✅  Supports MariaDB, PostgreSQL or H2. API is consistent across backends | ✅⭐️ Many supported backends however API features vary across backends; Active Directory, AWS, Azure, Consul, Cubbyhole, Databases, Google Cloud, Key/Value, Identity, Nomad, PKI (Certificates), RabbitMQ, SSH, TOTP, Transit
| Hardware encryption support | ✅⭐️ Supported in Open Source | ⚠️ Enterprise version required |
| Authentication Mechanisms | <ul align="left"><li>Mutual TLS</li><li>Cloudfoundry UAA OAuth 2 Password Grant</li><li>Cloudfoundry UAA OAuth 2 Client Credentials Grant</li></ul>|<ul align="left"><li>Tokens</li><Mutual TLS</li><li>JWT/OIDC</li><li>AppRole</li><li>AWS IAM</li><li>Azure Active Directory</li><li>Google Cloud</li><li>Github</li><li>Okta</li><li>RADIUS</li><li>Kubernetes</li><li>LDAP</li>|
| Secret typing / validation | ✅⭐️ [Secrets have types such as `user`, `certificate` and `json`](https://docs.cloudfoundry.org/credhub/credential-types.html) ) | ✅ The underlying secrets engine is able to enforce typing of secrets but not an explicitly stated goal of Vault |
| Credential generation with rules | ✅⭐️[Password complexity rules and certificate attribute configuration](https://credhub-api.cfapps.io/#generate-credentials)| ⚠️ Only some secret backends support credential generation. Rules not configurable by the Vault API but in theory the secret backend could enforce rules |
| Credential Versioning |  ✅ [Can retreive last `n` versions of a credential](https://credhub-api.cfapps.io/#get-by-name) | ✅ [Since v0.10](https://www.hashicorp.com/blog/vault-0-10) |
| Hierachical access control lists  |  ✅ [Documentation limited at present](https://credhub-api.cfapps.io/#permissions)  | ✅⭐️[Vault Policies](https://www.vaultproject.io/docs/concepts/policies.html) |
| Usage auditing | ✅ [Limited docs](https://github.com/cloudfoundry-incubator/credhub/blob/master/docs/product-summary.md)|✅⭐️ [Auditing to file, syslog or socket](https://www.vaultproject.io/docs/audit/index.html)|
| High availability | ✅ [When deployed as a service](https://github.com/pivotal-cf/credhub-release/tree/master/docs) | ✅ [Large number of deployment scenarios](https://www.vaultproject.io/docs/concepts/ha.html)|
| DevOps tool integration | ⚠️ Main focus is BOSH and Concourse| ✅⭐️ Most major tools have mature integration including [Ansible](https://docs.ansible.com/ansible/2.5/plugins/lookup/hashi_vault.html), [Chef](https://docs.chef.io/chef_vault.html), [Puppet](https://github.com/jsok/puppet-vault) and [Concourse](https://concourse-ci.org/creds.html#vault). There is currently no stable Vault integration as BOSH config server. |
| Spring Cloud Config Server integration | ⚠️ Currently not but in the pipeline | ✅ [Spring Cloud Config Server - Vault Backend](https://cloud.spring.io/spring-cloud-config/single/spring-cloud-config.html#vault-backend) |
| Spring Boot integration | ✅ [Spring Cloud Credhub](https://spring.io/projects/spring-credhub)| ✅ [Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)| 
| One-Time use secrets / self destructing secrets|⛔️|✅ Implemented with [Cubbyhole Response Wrapping](https://www.vaultproject.io/guides/secret-mgmt/cubbyhole.html)|


### Which tool to use?
CredHub is deeply integrated with:

* [Cloud Foundry Application Runtime](https://www.cloudfoundry.org/application-runtime/) neé Elastic Runtime
* [Cloud Foundry Container Runtime] (https://www.cloudfoundry.org/container-runtime/) neé Kubo - Kubernetes powered by BOSH
* [BOSH](https://www.cloudfoundry.org/bosh/). 

If you are using any of these platforms you are probably already using CredHub to some degree so you should strongly consider exploiting CredHub's capabilities to the maximum! <mark>In this scenario it is also worth considering synergising CredHub with a Vault installation</mark> 

Using CredHub in the absence of Cloud Foundry, CFCR or BOSH is certainly possible but it is not optimized for this use case and you will probably find Vault to be more appropriate for your needs.
## A Belt and Braces approach:<br />CredHub + Vault + Spring = (🔒 * ❤️)
{{< figure src="https://i.imgur.com/iUpB8WE.jpg" >}}
Deploying both CredHub & Vault allows us to combine the strengths of each product and maximise protection of credentials.