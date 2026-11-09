# Experimental scan-group reverse proxy

`nginx.conf` serves the branch-built GSA bundle and protects every
`/credential-api/` request with an internal GSAD authorization request to
`/scan-group-auth`. GSAD validates the session cookie, token, expiry, and
client address, then permits only the session whose server-side username
matches `SCAN_GROUP_GMP_USERNAME`. Set it to the same account as the
controller's `GMP_USERNAME`; that account can see all review groups. Invalid
sessions receive 401, other valid sessions receive 403. The setting is
required and fails closed if missing.

After authorization, nginx reaches the controller through the private
`/run/credential-review/controller.sock` Unix socket. Mount its volume only
in nginx and the controller, set `TRUST_GSA_PROXY=1` and
`REVIEW_HTTP_SOCKET=/run/credential-review/controller.sock`, and publish no
controller TCP port. The browser uses its existing GSA session without an
operator password prompt.

This proxy and companion controller are the current review path. The
standalone C wire proposals remain for a future native GMP implementation.
