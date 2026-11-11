# Authentication options on Server

Type: research (AFK)
Status: resolved
Label: wayfinder:research

## Question

Which authentication options can an unattended read-only service use against each Server version and Services (personal access token, Windows integrated authentication under a service account, anything else), and what is the least-privilege read-only setup for each? What identity data do the APIs return?

Findings file: [docs/research/03-authentication-options-on-server.md](../../../docs/research/03-authentication-options-on-server.md)

## Answer

2026-11-11. From Microsoft documentation only; nothing was tested against a live server. Detail and sources are in the findings file above.

- On every Server version in scope, an unattended client can use a personal access token or Windows authentication (NTLM or Kerberos through Negotiate).
- Microsoft Entra tokens, service principals, managed identities and OAuth apps are documented for Services only, not for Server.
- On Services, Microsoft steers new integrations to Entra tokens and treats personal access tokens as the highest-risk choice. Tokens still work there but tenant policies can block or shorten them.
- A token never grants more than its owner has, so least privilege has two layers: a dedicated identity in the Readers group, and a token limited to read scopes. Microsoft defines no minimal permission set for reporting tools.
- The published scope list has no read-only scope for pipeline environments; the Environments List API names only a manage scope. This matters for v0.2.
- Windows authentication from a Linux container is not covered by the Azure DevOps documentation.
- Identity data arrives in several shapes (display name, unique name that is often an email or DOMAIN\user, id, descriptor), and service identities appear alongside people.

Still unverified: maximum token lifetime, which scopes each Server version offers, whether Graph endpoints exist on Server, service accounts from Linux containers, and anything published only on the Azure DevOps blog, which was blocked from the research environment.
