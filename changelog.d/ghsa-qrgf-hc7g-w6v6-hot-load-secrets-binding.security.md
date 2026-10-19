**core:** a server loaded from the registry with `hangar_load` is bound to
what the operator allows it, not to what its registry entry asks for.

- **Secrets.** The variables an entry names were resolved from the
  gateway's own environment and from the secrets directory, and delivered to
  the launched server, so an entry asking for `AWS_SECRET_ACCESS_KEY`
  received the gateway's credential. Only the names listed for the server's
  exact registry name in the new `hot_loading.secrets.allow` are now looked
  up; any other is reported in `missing_secrets`, naming the key and the
  server. A name that is not an environment variable name is refused before
  anything is consulted, every segment of the server's directory is checked,
  and a file is read only when its resolved path is inside the secrets
  directory, so `../` in a name or a server and a symlink out of the
  directory read nothing. Allowlist keys that differ only in case are
  refused.
- **Environment.** A hot-loaded server inherited the gateway's environment
  minus a name heuristic, so `AWS_ACCESS_KEY_ID`, webhook URLs and DSNs
  reached it. Hangar now passes it only a fixed base (`PATH`, `HOME`,
  locale, timezone, temporary directory, and the Windows equivalents) and
  the names in the new `hot_loading.inherit_env`. Statically configured
  servers are unchanged. What Hangar hands the server is bounded; a
  hot-loaded server is code running as the gateway's user, so
  `trusted_namespaces` is the boundary.
- **Verified.** The gate tested the registry's `status: active`, which
  every published entry carries. An entry is now verified when its registry
  name starts with a namespace in the new `hot_loading.trusted_namespaces`,
  matched case-sensitively by whole segment, and the registry lists it as
  active. Empty, the default, verifies nothing. `ServerSummary.is_official`
  and `ServerDetails.is_official` are renamed `is_published`.
- **force_unverified.** The `hangar_load` argument is the caller's, and the
  tool's own answer told the caller to set it. It is now ignored unless the
  operator sets the new `hot_loading.allow_force_unverified: true`, and the
  answer names the operator's keys instead.
- **Echoed text.** An entry whose name has whitespace or a control
  character is treated as not found, and every name echoed from an entry is
  escaped, so a name cannot add lines to what the agent reads. A variable
  that is not bound gets "Ask the operator to allow it" in prose, never a
  ready-to-paste config grant.
