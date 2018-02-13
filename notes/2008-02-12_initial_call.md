# Initial kick off call

## Attendees:
* Michael Hausenblas
* Evan Gilman

## Notes:
* Dev / Testing should be pretty straight forward with Spire uisng the Docker based workflow
* Once the agents starts correctly the SVID is output to disk, this can then be used to auth against Vault
* Every service requires a Spire registration
* Possible that there may be multiple trust domains
* The service may not be aware of it's Trust Domain
* Vault would require the CA for each Trust Domain in order to validate the SVID passed at auth
* Policy should be mapped at both a `Trust Domain` and `Path` level allowing configuration of default domain policy with the additional benefit of enhancing or degrading permissions at a path level.  This is in line with other Auth backends.

## Questions:
The main question around implementation is should a SPIFFE auth endpoint be mounted for each Trust Domain or on a higher level.  The main consideration around this is that the service would potentially not know it's Trust Domain without decoding the SPIFFE ID and would need to call a custom endpoint based on this result.  This would lead to a better UX flow where a single auth endpoint is mounted and Vault would infer the Trust Domain from the SPIFFE ID encoded into the SVID.  
From further reading it seems that the Trust Domain into which a service is placed would be known by the operator and therfore the Vault Auth Path could be provided to the service as configuration.  Having a 1-1 mapping between Trust Domain and Vault Auth endpoint would be more inline with other auth methods such as the github auth.