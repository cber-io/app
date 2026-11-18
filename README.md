# App domain configuration

`config.json` is the existing Cber App bootstrap payload. `orgId` remains a string;
`responseRandomKey` and the AES-128-ECB/PKCS7 encoded `domains` retain their existing
format. The decrypted shape remains `{"normal":[{"domain":"example.com","level":1}]}`.
Node order and `level` are meaningful and must be preserved.

The optional outer field `update_at` is the domain-list version: a JSON integer
containing UTC Unix epoch milliseconds. It is the configuration publication time,
not the time of a request, CDN response, deployment or client fetch. The same
content revision must use the same value in origin APIs and this file.
Adding this field does not require existing clients to change their parsers or
selection logic. Missing `update_at` means an unversioned legacy response.

New clients can request approved GitHub/raw, jsDelivr and origin endpoints in
parallel. First validate the organization, decryption, node list and timestamp,
then atomically persist the newest valid version together with its domains.
Never let a late older/unversioned response replace versioned local data. An equal
version with different content is a conflict, not a tie to resolve by arrival time.
Version comparison selects the configuration; keep existing line speed tests,
`level` interpretation and active-line selection behavior separate.

Every actual content change (including order, level or rollback) requires a new
timestamp strictly greater than the last published version. Retries, cache purges,
retagging and deployments must not advance it. Do not change the ciphertext while
retaining an old `update_at`. Use the coordinated publication workflow rather
than editing `config.json` by hand.

`main` is consumed directly by raw GitHub clients. jsDelivr `@latest` also requires
a new stable semver tag; do not move an existing tag. Old CDN and device cache
responses may coexist during rollout, so timestamp comparison does not replace
cache refresh policy. See [jsDelivr documentation](https://www.jsdelivr.com/documentation).

The initial versioned payload retains the complete existing encrypted domain
list; only `update_at` is added. `config.schema.json` describes the additive
contract and is not injected into client responses.
