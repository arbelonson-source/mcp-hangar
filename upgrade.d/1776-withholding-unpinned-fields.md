### a pinned tool is served without its title, annotations, icons and `_meta`

A tool under a digest pin in `block` mode (the default `digest_enforcement`)
is now served with only `name`, `description`, `inputSchema` and
`outputSchema`, the fields the pin covers. Its `title`, `annotations`,
`execution`, `icons` and `_meta` are left out of the front door's
`tools/list`, `server/discover`, `hangar_tools`, `hangar_details` and the REST
server and tool routes. Hangar's own fields (`digest`, `pinned_digest`,
`withheld`) are unchanged. The cut does not depend on where the fields came
from: a `title` or `annotations` you wrote yourself for a server's predefined
`tools` in the configuration is stripped too while the tool is under a `block`
pin.

What a client sees:

- The display name falls back to `name`.
- With no `annotations`, the spec's defaults apply: `readOnlyHint: false`,
  `destructiveHint: true`, `idempotentHint: false`, `openWorldHint: true`. A
  client that skipped confirmation for a pinned read-only tool will ask again.
- `execution` is left out on every protocol version, but only a handshake-era
  client (2025-11-25 and earlier) reads `execution.taskSupport` to decide
  whether to call a tool as a task. Such a client no longer sees a pinned tool
  that declared `taskSupport: "required"` as a task tool, and calls it without
  one. On 2026-07-28, task use is negotiated through the tasks extension
  instead.

What to do:

- Nothing, if the pinned tools' metadata is not load-bearing for your clients.
- A tool left unpinned, or a server pinned with `digest_enforcement: warn` or
  `audit`, is served whole as before. Both also give up the protection: an
  unpinned tool is not checked, and `warn`/`audit` let a drifted call through.
  A follow-up is planned to let a pin approve these fields explicitly.
- The digest is unchanged: existing pins stay valid and need no `pin --write`.
