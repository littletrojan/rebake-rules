# rebake-rules

Validation rule packs for [Rebake](https://github.com/) — the AI-native
BIM tool. This repo is the upstream source of truth for Rebake's
built-in IFC compliance rules and the schema third parties use to
ship their own packs.

## What lives here

```
packs/                  JSON rule packs loaded by Rebake at startup.
  ifc-classes-v1.json     Allowlist of valid IFC 4.3 entity classes
                          per Rebake element kind. The default pack.
schema/
  rule-pack.schema.json   JSON Schema (draft 2020-12) every pack must
                          validate against.
LICENSE                 MIT.
README.md               You are here.
```

## How Rebake consumes it

Rebake pins this repo as a git submodule. At build time the main
repo's CMake step copies `packs/*.json` into `bin/rules/` alongside
`rebake_c.dll`. At app launch, Rebake's validator scans:

1. `bin/rules/`               — built-in packs (this repo's contents).
2. `%APPDATA%\Rebake\rules\`  — user-global packs.
3. The opened project's local packs (when project-local rule packs
   land — TBD).

All three paths use the same loader, the same schema, and the same
rule format. Built-ins are not a special code path; they're just the
packs that ship in the install.

## Pack format (v1)

Every pack is a single JSON object with `schema_version`, a stable
machine `id`, a semver `version`, a human-readable `name`, and an
optional `description`. Rule data lives in arrays keyed by rule kind.
Today's only rule kind is `allowlist`:

```json
{
  "schema_version": 1,
  "id": "rebake-builtin-ifc-classes",
  "version": "1.0.0",
  "name": "Rebake built-in IFC classes (v1)",
  "allowlist": [
    {
      "applies_to": "wall",
      "allowed": ["IfcWall", "IfcWallStandardCase", "IfcCurtainWall"]
    }
  ]
}
```

`schema/rule-pack.schema.json` is authoritative — if your pack passes
that schema, Rebake will load it.

Warnings cite the rule by `pack_id::rule_id` so a violation always
points back to a real file in this (or a user's) repo.

## Adding a new IFC class

Open `packs/ifc-classes-v1.json`, add the entity name to the right
`allowed` array, bump the pack `version` (semver patch is fine for
purely additive changes), commit, push. Rebake picks it up on the
next submodule pin bump in the main repo.

## License

MIT — fork it, ship your own firm-specific rule pack on top, no
permission needed. See `LICENSE`.
