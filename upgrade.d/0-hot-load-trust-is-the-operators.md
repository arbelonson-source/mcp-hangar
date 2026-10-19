### `hangar_load` loads only what the operator trusts, with only the environment and secrets bound to it

Before, any server published in the registry could be loaded with
`hangar_load`, because the registry marks every published entry
`status: active` and that status was read as "verified". The variables its
entry named were looked up in the gateway's environment and in the secrets
directory and passed to it. It also inherited the gateway's whole environment,
minus the names a heuristic called sensitive. A caller could load an entry
that was not active by passing `force_unverified=True`.

Now four `hot_loading` keys decide, and each defaults to refusing:

- `trusted_namespaces`: a list of registry name prefixes. An entry is
  verified when its name is under one of them **and** the registry lists it
  as active. Anything else is refused with `status: unverified`. With the
  default, an empty list, every `hangar_load` is refused until a namespace is
  listed. Every namespace ends in `/`, one is added if it is left out, and
  matching is case-sensitive by whole segment, so list each sub-namespace you
  trust.
- `allow_force_unverified`: `false` by default, and `force_unverified` is
  then ignored. Set it to `true` only to let callers load entries that are
  not verified, a deprecated entry in a trusted namespace included.
- `secrets.allow`: the exact registry name of a server mapped to the
  variables it may receive. A variable not listed is answered with
  `status: missing_secrets`, even when the gateway has it set. Two keys that
  differ only in case are refused.
- `inherit_env`: names a hot-loaded server takes from the gateway's
  environment, in addition to a fixed base: `PATH`, `HOME`, `USER`,
  `LOGNAME`, `LANG`, `LC_ALL`, `LC_CTYPE`, `TZ`, `TMPDIR`, `TMP`, `TEMP`,
  and on Windows `SYSTEMROOT`, `COMSPEC`, `PATHEXT`, `APPDATA`,
  `LOCALAPPDATA`, `USERPROFILE`. Hangar passes it nothing else from its
  environment. A server that
  needs a proxy (`HTTPS_PROXY`), a CA bundle (`SSL_CERT_FILE`) or `UV_*` and
  `npm_config_*` settings to install needs them listed here. Statically
  configured servers inherit as before.

What Hangar hands the server is bounded; a hot-loaded server is code running
as the gateway's user, so `trusted_namespaces` is the boundary. It can read
whatever files that user can, the secrets directory and `~/.aws/credentials`
included, on its own.

To keep loading a server as before, add its namespace, its variables and
anything its runtime needs from the environment:

```yaml
hot_loading:
  trusted_namespaces: ["io.github.acme/"]
  inherit_env: [HTTPS_PROXY]
  secrets:
    allow:
      "io.github.acme/weather": [WEATHER_API_KEY]
```

A per-server secrets file is still read from
`~/.config/mcp-hangar/secrets/<registry name>/<NAME>`, one directory per `/`
segment of the name (`secrets/io.github.acme/weather/WEATHER_API_KEY`). A
name with a segment outside `[A-Za-z0-9._-]`, or equal to `.` or `..`, has no
per-server directory. A variable name that is not an environment variable
name, or a file whose resolved path leaves the secrets directory, reads
nothing.

#### Changed signatures

- `ServerSummary.is_official` and `ServerDetails.is_official` are now
  `is_published`. It means only that the registry lists the entry as active.
- `SecretsResolver.resolve(required, mcp_server_name)` is now
  `SecretsResolver.resolve(required, mcp_server_name, *, allowed)`: `allowed`,
  the names the server may receive, is a required keyword argument.
- `SecretsResolver.get_missing_instructions(missing: list[str], mcp_server_name)`
  is now `get_missing_instructions(result: SecretsResult, mcp_server_name)`.
- `LoadMcpServerHandler` takes `trust`, a callable returning the
  `HotLoadTrust` in force. Omitted, nothing is trusted or delivered.
- `UnverifiedMcpServerError` takes the keyword arguments `force_ignored`,
  `published` and `trusted`, and its message no longer suggests
  `force_unverified`.
