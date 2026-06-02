# General Usage

Users will primarily interact with this software through the `DataHandler` class. By default, Solar Data
Tools uses CLARABEL as the solver for all signal decomposition problems. If you would like
to specify a solver (such as MOSEK), just pass the keyword argument `solver` to `DataHandler.pipeline` with the solver of choice.

The data should be in the form of a pandas DataFrame with a datetime index and a column for the power signal
(or the user must set the `datetime_col` kwarg.) The data is recommended to be in the local timezone of the PV system.

```python
import pandas as pd
from solardatatools import DataHandler

pv_system_data = pd.read_csv('path/to/your/data.csv', index_col=0, parse_dates=True)

dh = DataHandler(pv_system_data)
dh.run_pipeline(power_col='dc_power')
```

If everything is working correctly, you should see something like the following:

```bash
total time: 25.99 seconds
--------------------------------
Breakdown
--------------------------------
Preprocessing              6.76s
Cleaning                   0.41s
Filtering/Summarizing      18.83s
    Data quality           0.21s
    Clear day detect       0.44s
    Clipping detect        15.51s
    Capacity change detect 2.67s
```

You can find more in-depth usage examples in the [demo](./notebooks/demo_default.nblink) and [tutorial](./notebooks/tutorial.ipynb) notebooks.
