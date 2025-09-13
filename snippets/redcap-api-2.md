Shortcomings of the REDCap API -  
Need for an API 2.0?
==============

## Limitations of the Current REDCap API

Back in the _Motivation_ and _Prerequisits_ sections, I mentioned the export, either via CSV download or API export, of **reports**.

Using reports is powerful, because this leaves full control over the exported data with the project designer/owners. However, currently the REDCap API only supports very coarse rights: **API Import** and **API Export**. Both, the `Export Records` and the `Export Reports` methods are available with API Export right, but they cannot be differentiated further, i.e., it is not possible to grant one without the other.

REDCap does not support field-level access rights. However, with the **combination of report access, data viewing, and data export rights**, in principle, **field-level granularity** could be achieved, _if only it were possible to limit API access to exporting reports_.

But there are more issues with the current implementation of the REDCap API and token management.

- API endpoint is shared between API, REDCap Mobile App, and MyCap.
- Only one token per user and project is possible.
- The same token is shared for API access and REDcap Mobile App operations
- In external authentication scenarios (LDAP, Shibboleth, OpenID, etc.), the lifetime of tokens is "uncoupled", i.e. a token can still be used when a user cannot log into REDCap any longer.
- API user right options (Import, Export, EM methods) allow only very coarse control of access.
- No management options to auto-expire tokens
- REDCap tokens do not observe modern security requirements

Several key areas for improving the API have been identified and discussed in previous REDCapCon session:

- **Tokens and token management**
  - Multiple tokens per user and project
  - Service tokens / Affiliate tokens (for external access; way to get a token to an external user)
  - "System tokens" (i.e., not associated with a particular project)
  - Token request pipeline (custom survey)
  - Granular, method-level restrictions
- **Modern security architecture**
  - Access and refresh tokens
  - Token expiry
- **Access restrictions**
  - IP allowlist (IP of originating calls)
  - Per-token rate limiting
- **Better logging**
  - Visualization / reporting
  - Performance insights

## Proposal of a Revised API 2.0

- **Built on JSON-RPC and OpenRPC**
  - JSON-RPC
    - A standard for exchanging requests/responses based on JSON  
      <https://www.jsonrpc.org/>
    - Currently at version 2.0
  - OpenRPC
    - A standard for documenting a JSON-RPC API  
      <https://open-rpc.org/>
    - Based on JSON, machine-readable
    - Can be used to automatically generate documentation as well as tooling
    - REDCap’s JSON-RPC API will be documented in OpenRPC and provide the `rpc.discover` method
- **Documentation-driven tooling** (based on OpenRPC)
  - Documentation and Playground, but
  - Auto-generated,
    - providing custom-tailored API documentation for any given project (and depending on the rights associated with the token), which
    - includes all project-specific items such as field, form, event names (if applicable), and
    - is available through the API itself (via `rpc.discover`).
  - This change will make it **much easier to introduce new API methods**, because documentation and the API Playground will no longer need to be implemented separately, a step that currently poses a major hurdle.
- **API versioning**
  - API versioning built-in (`/api/?json-rpc=v1`)
  - Similar to EM Framework (new versions only when there are breaking changes to existing methods)
  - New methods will always be added to v1 (increasing the minor version; minor version number will be the same for all major versions)
  - This provides easy means for deprecation and retirement of methods
- **New permissions model**
  - **_Method-level_ granularity** (for all built-in and EM-provided methods)
  - JSON-RPC API access and the (maximally) granted methods (scope) are set through user rights (roles, individual user rights)
  - Global API settings
    - Only allow certain originating IP addresses
    - Global default rate limits
  - Other security options / limitations are set on a **per-token** basis
- **Improved token management**
  - New token management system
    - Legacy API tokens will **not** work with the new API
  - Users can generate **as many tokens as they like** (and they should!)
  - Tokens can be **limited in scope** (allowed methods; the maximal scope is define by the user’s rights)
  - Tokens can be recycled, deleted, transferred (as now)
  - Tokens, _individually_, can be assigned 
    - an **expiration date**, 
    - a **maximum session duration**, 
    - an **allowed IP range**, and 
    - a **rate limit**.
- **Support for asynchronous file transfers**
- **Improved security**
  - Encouragement of _service-specific_ API tokens
  - API tokens are only used to start an API session. The `sessionStart` method grants a set of session tokens:
    - Access token
      - To be used for all subsequent requests
      - Short expiration (in the minutes to few hours range)
    - Refresh tokens
      - Longer lifetime (weeks to months)
      - Used to refresh a session once the access token has expired – provides new set of access and refresh token
    - Sessions can therefore potentially last indefinitely if tokens are recycled in time
  - An API token can start any number of (concurrent) sessions
  - Limited session scope (allowed methods can further be limited through the client at the start of the session and cannot be changed for the duration of this session)
  - Session termination (clients can choose to end their current session or all sessions associated with the API token used to start the session)
  - OAUTH 2.0 recommended access/refresh-token management
    - Refresh token can only be used once
    - In case of suspected compromise (attempted second use of a refresh token, use of a refresh token without the last valid access token), all sessions initiated with the API token are terminated
    - Logging of such illegal events

