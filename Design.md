# Imeplementation Design
This document will contain the initial implementation design.


## Example Trust Domains
The below tables show some example SPIFFE trust domains and how they may map to a Vault cluster and auth point, the current examples would validate based on and individual trust domains CA, there is currently no concept for hierarchical trust domains and validation based on trust chain.

### Simple example where an organisation has a single trust domain per environment
It is generally good practice to isolate secrets across environments, for example a developer may have different levels of permissions on a Dev cluster which does not carry production data.  Staging would most likely not be using production datastores or other elements however depending on the purpose of the Staging environment, i.e. should it carry sensitive or regulated infomation then it would most likely have tighter access control.  If Staging uses dummy or non-sensitive data then it could potentially share the development Vault cluster.

| Trust Domain            | Vault Cluster | Auth Mount Point |
| ----------------------- | ------------- | ---------------- |
| spiffe://prod.acme.net/ | Production    | /v1/auth/spiffe  |
| spiffe://stg.acme.net/  | Staging       | /v1/auth/spiffe  |
| spiffe://dev.acme.net/  | Development   | /v1/auth/spiffe  |
  
  
### Example showing `spiffe://prod.acme.net/` as a global identity and a trust domain per geo location.
A large geo-distributed application would most likely run Vault premium which allows multi-datacenter performance replication.  In this instance a Vault Cluster would exist in each Region however the Clusters would be joined and in terms or read and writes would share the same data.

| Trust Domain               | Vault Cluster | Auth Mount Point   | Comments |
| -------------------------- | ------------- | ------------------ | ------------------------------------------ |
| spiffe://us.prod.acme.net/ | Production    | /v1/auth/spiffe/us | Vault premium with performance replication | 
| spiffe://eu.prod.acme.net/ | Production    | /v1/auth/spiffe/eu | "" |
| spiffe://ap.prod.acme.net/ | Production    | /v1/auth/spiffe/ap | "" |


### Trust domain per department or application boundary
In the below example, the two trust domains `insurance` and `consumer` would most probably share the same cluster in an enterprise.  However, Support may or may not use it's own cluster, ideally support would require access to secrets such a Database Users, AWS credentials, etc, therefore it would make sense to allow access to the main Vault cluster instead of having to replicate and maintain secrets in two clusters.  Policy in Vault would allow for privilege to be restricted to the right levels ensuring any sensitive information which support are not allowed to access remains secret.  The organisation would most likely leverage Vault premium's capability to run in more than one datacenter. If the organisation was particularly security adverse then they may use their own infra for support application secrets and allow support personnel to auth the production Vault cluster to obtain the secrets required to solve problems.

| Trust Domain                                  | Vault Cluster  | Auth Mount Point         | Comments  |
| --------------------------------------------- | -------------- | ------------------------ | --------- |
| spiffe://insurance.bigbank.com/               | Production     | v1/auth/spiffe/insurance |           |
| spiffe://consumer.bigbank.com/                | Production     | v1/auth/spiffe/consumer  |           |
| spiffe://support-site.prod.consumer.acme.net/ | Production?    | v1/auth/spiffe/support   | support-site has dedicated infra |


## Questions / Assumptions
* SPIFFE allows a Trust Domain to be signed by a parent Domain forming a chain of trust, it is assumed that Vault would validate an SVID based on the full CA bundle and would NOT validate a leaf node based on the Root CA.