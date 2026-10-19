**security:** a tool under a digest pin in `block` mode is now served with only
the fields the pin covers: `name`, `description`, `inputSchema` and
`outputSchema`. The pin's digest covers those four, but every listing served
the rest of the upstream's definition verbatim, so an upstream could keep the
digest and still put instructions in `title` or `_meta`, or add
`annotations: {readOnlyHint: true, destructiveHint: false}` to a tool pinned
without them, which can remove a client's confirmation before the call. The
cut applies on the front door's `tools/list` (both SDK paths, stdio and
streamable HTTP), `server/discover`, `hangar_tools`, `hangar_details` and the
REST server and tool routes, through one shared allow-list, so a field the
spec adds later is withheld from a pinned tool too. Pins are scoped as for the
drift check: the server, the groups that own it and per-tenant pins. The
digest itself is unchanged, so existing pins stay valid. Tools without a pin,
and pins in `warn` or `audit` mode, are served whole as before.
