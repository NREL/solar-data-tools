# Archival Artifacts

This directory holds files retained for historical reference only. They are
**non-functional**: they depended on modules that have been removed from Solar
Data Tools and will not execute.

The directory structure mirrors each artifact's original location in the
repository:

- `tests/fixtures/` — Jupyter notebooks that were historically used to generate
  the CSV test fixture data via the deprecated `dataio` module (removed in
  version 2.0). The CSV fixtures themselves are still in active use and remain
  in the top-level `tests/fixtures/` directory.
- `sdt_dask/examples/dev_scripts/` — a development script that depended on
  `PVDBPlug`, a data plug that was removed along with `dataio`.

When retiring other non-functional artifacts in the future, move them here with
`git mv`, mirroring their original path, and document them in a `README.md`.
