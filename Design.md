# Implementation Design
This document contains the high-level implementation design.

## SPIFFE Identity
[SPIFFE Identity Documentation](https://github.com/spiffe/spiffe/blob/master/standards/SPIFFE-ID.md#2-spiffe-identity)

The SPIFFE ID contained within the SPIFFE Document (SVID) is represented by the URI syntax `spiffe://[trust_domain]/[path]`. The Trust Domain is a required element and represents the top level of trust for a logical separation.  Path is the element which represents the unique name for the application or service and follows a canonical URI path.  The structure of the path is determined by the operator of the system and paths may be used to identify sub divisions within a service.  Operators can choose to build SPIFFE IDs in a number of ways, for example the payments queue in the following examples could be constructed by adding a Trust Domain which is a leaf on the prod trust domain or it may be constructed as a path on the prod trust domain.

| SPIFFE ID                                   | Trust Domain  | Application ID       | Comments                                           |
| ------------------------------------------- | ------------- | -------------------- | -------------------------------------------------- |
| spiffe://prod.acme.net/123-123d-sdfs3       | prod.acme.net | 123-123d-sdfs3       | Generic service identifier using GUID              |
| spiffe://prod.acme.net/currency/mysql       | prod.acme.net | currency/mysql       | MySQL database for the currency service            |
| spiffe://prod.acme.net/currency/web         | prod.acme.net | currency/frontend    | Application Frontend for the currency service      |
| spiffe://payments.prod.acme.net/email/queue | payments.prod.acme.net | email/queue | Queue for the email service in the payments domain | 
| spiffe://prod.acme.net/currency/web         | prod.acme.net | payments/email/queue | Queue for the email service in the payments domain | 

### Trust Domains
The below tables show some example SPIFFE trust domains and how they may map to a Vault cluster and auth point, the current examples would validate based on and individual trust domains CA, there is currently no concept for hierarchical trust domains and validation based on trust chain.  
**Trust Domains will map to Vault SPIFFE Auth mount points as a 1-1 relationship**

#### Simple example where an organization has a single trust domain per environment
It is generally good practice to isolate secrets across environments, for example a developer may have different levels of permissions on a Dev cluster which does not carry production data.  Staging would most likely not be using production datastores or other elements however depending on the purpose of the Staging environment, i.e. should it carry sensitive or regulated information then it would most likely have tighter access control.  If Staging uses dummy or non-sensitive data then it could potentially share the development Vault cluster.

| Trust Domain            | Vault Cluster | Auth Mount Point |
| ----------------------- | ------------- | ---------------- |
| spiffe://prod.acme.net/ | Production    | /v1/auth/spiffe  |
| spiffe://stg.acme.net/  | Staging       | /v1/auth/spiffe  |
| spiffe://dev.acme.net/  | Development   | /v1/auth/spiffe  |
  
  
#### Example showing `spiffe://prod.acme.net/` as a global identity and a trust domain per geo location.
A large geo-distributed application would most likely run Vault premium which allows multi-datacenter performance replication.  In this instance a Vault Cluster would exist in each Region however the Clusters would be joined and in terms or read and writes would share the same data.

| Trust Domain               | Vault Cluster | Auth Mount Point   | Comments |
| -------------------------- | ------------- | ------------------ | ------------------------------------------ |
| spiffe://us.prod.acme.net/ | Production    | /v1/auth/spiffe/us | Vault premium with performance replication | 
| spiffe://eu.prod.acme.net/ | Production    | /v1/auth/spiffe/eu | "" |
| spiffe://ap.prod.acme.net/ | Production    | /v1/auth/spiffe/ap | "" |


#### Trust domain per department or application boundary
In the below example, the two trust domains `insurance` and `consumer` would most probably share the same cluster in an enterprise.  However, Support may or may not use it's own cluster, ideally support would require access to secrets such a Database Users, AWS credentials, etc, therefore it would make sense to allow access to the main Vault cluster instead of having to replicate and maintain secrets in two clusters.  Policy in Vault would allow for privilege to be restricted to the right levels ensuring any sensitive information which support are not allowed to access remains secret.  The organization would most likely leverage Vault premium's capability to run in more than one datacenter. If the organization was particularly security adverse then they may use their own infra for support application secrets and allow support personnel to auth the production Vault cluster to obtain the secrets required to solve problems.

| Trust Domain                                     | Vault Cluster  | Auth Mount Point         | Comments  |
| ------------------------------------------------ | -------------- | ------------------------ | --------- |
| spiffe://insurance.bigbank.com/                  | Production     | v1/auth/spiffe/insurance |           |
| spiffe://consumer.bigbank.com/                   | Production     | v1/auth/spiffe/consumer  |           |
| spiffe://support-site.prod.consumer.bigbank.com/ | Production?    | v1/auth/spiffe/support   | support-site has dedicated infra |

## Vault Identity Mapping 
[Vault Identity Secrets Engine Documentation](https://www.vaultproject.io/docs/secrets/identity/index.html)
Given the possible structure combinations as seen in the examples of SPIFFE IDs and Trust Domains Vault Identity Mappings will translate to the following simplified levels:

| SPIFFE       | Vault  |
| ------------ | ------ |
| Trust Domain | [Group](https://www.vaultproject.io/api/secret/identity/entity.html)  |
| Path         | [Entity](https://www.vaultproject.io/api/secret/identity/group.html)  |

This will allow the operator to assign policy to a Trust Domain level and additionally have finer grain control on an application or service level, following standard Vault policy assignment Entity level policy could either be added or subtracted from the Group level.

| SPIFFE ID                                | Vault Group      | Vault Entity       |
| ---------------------------------------- | ---------------- | ------------------ |
| spiffe://prod.acme.net/currency/web      | prod.acme.net    | currency/frontend  |
| spiffe://staging.acme.net/currency/web   | staging.acme.net | currency/frontend  |
| spiffe://prod.acme.net/123-123d-sdfs3    | prod.acme.net    | 123-123d-sdfs3     |

## Vault Auth Endpoint Mapping
As previously mentioned in this document Auth Mount points will be mapped to Trust Domains as a **1-1** relationship

## Authentication Flow
**TODO**

## Assumptions
* SPIFFE allows a Trust Domain to be signed by a parent Domain forming a chain of trust, it is assumed that Vault would validate an SVID based on the full CA bundle and would **NOT** validate a leaf node based on the Root CA.
* Vault will consider a SPIFFE ID Path as a unique entity per Trust Domain (SPIFFE Auth Mount) and will not consider sub paths.

## References:
* [Vault Identity (Entity)](https://www.vaultproject.io/api/secret/identity/entity.html)
* [Vault Identity (Group)](https://www.vaultproject.io/api/secret/identity/group.html)
* [SPIFFE Identity](http://github.com/spiffe/spiffe/blob/master/standards/SPIFFE-ID.md#2-spiffe-identity)
* [SPIFFE Verifiable Identity Document](https://github.com/spiffe/spiffe/blob/master/standards/SPIFFE-ID.md#3-spiffe-verifiable-identity-document)
