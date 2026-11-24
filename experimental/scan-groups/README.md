# Experimental scan-group reverse proxy

`nginx.conf` serves the branch-built GSA bundle and protects every
`/credential-api/` request with an internal GSAD authorization request to
`/scan-group-auth`. GSAD validates the session cookie, token, expiry, and
client address, then permits only the session whose server-side username
matches `SCAN_GROUP_GMP_USERNAME`. Set it to the same account as the
controller's `GMP_USERNAME`; that account can see all review groups. Invalid
sessions receive 401, other valid sessions receive 403. The setting is
required and fails closed if missing.

The exact `/credential-api/groups/task-membership` route uses the same
administrator check. It accepts at most 100 supplied task IDs and returns
only which are controller-managed. Non-admin sessions receive 403, including
when they supply a private managed task UUID. GSA preserves individual and
selected bulk controls for tasks owned by those users. Shared tasks remain
guarded when controller ownership cannot be verified.

After authorization, nginx reaches the controller through the private
`/run/credential-review/controller.sock` Unix socket. Mount its volume only
in nginx and the controller, set `TRUST_GSA_PROXY=1` and
`REVIEW_HTTP_SOCKET=/run/credential-review/controller.sock`, and publish no
controller TCP port. The browser uses its existing GSA session without an
operator password prompt.

This proxy remains the authorization boundary in the native review path. With
`NATIVE_BACKEND=1`, the companion controller calls gvmd's native multi-target
GMP commands while keeping credential planning and bounded combined-report
publication. The proxy does not need to inspect or schedule child scans.
