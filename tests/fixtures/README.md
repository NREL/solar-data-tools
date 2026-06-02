# Test Fixtures

The CSV files in this directory are used as inputs for the test suite.

## Notebooks

The Jupyter notebooks retained in several subdirectories below are **archival only**.
They were historically used to generate the CSV fixture data from the now-removed `dataio` module.
Because `dataio` was fully deprecated (all functions raised "no longer supported" errors),
these notebooks will not execute and are kept solely as documentation of how the fixture data was created.

Do not attempt to run them. To regenerate fixture data, load your own data source via `pandas.read_csv`
or another data-loading method and follow the notebook's processing steps manually.
