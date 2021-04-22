# Downloading ERA5 Data and creating ERA5 Cutouts

A short guide to downloading ERA5 data from the [Copernicus Data Store](https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-single-levels?tab=overview).

## Download data using geodata

To start, import the required dependencies:

```
import logging
logging.basicConfig(level=logging.INFO)
import geodata
```

`import geodata` is required to use **geodata**, while launching a logger allows for detailed debugging via the console.


```
DS_monthly = geodata.Dataset(module="era5",
					 years=slice(2015, 2015),
					 months=slice(1,12),
           weather_data_config = "wind_solar_monthly")
```

If the specified dataset does not yet exist, this will return the number of files left to download. You may optionally pass a `bounds` parameter to download a geographic subset of data (default is the entire world).

The data download can be initiated via the `get_data()` command:

```
if DS_monthly.prepared == False:
	DS_monthly.get_data()
```

Depending on your specified time range and internet connection speed, this step could take several minutes or even hours.

**Note: the ERA5 API frequently has long queues as requests are processed, with relatively rapid download rates once download commences. To save you time (and the ERA5 servers multiple requests), we recommend downloading the largest geographic bounds you may want.**



## Preparing the cutout

A cutout is the basis for any data or analysis output by the **geodata** package.  Cutouts are stored in the directory `cutout_dir` configured in `config.py` (to set up `config.py`, [see here](https://github.com/east-winds/geodata/blob/master/doc/general/packagesetup.md)).

```
cutout = geodata.Cutout(name="era5-europe-test-2011-01",
                       module="era5",
                       weather_data_config="wind_solar_monthly",
                       xs=slice(30, 41.56244222),
                       ys=slice(33.56459975, 35),
                       years=slice(2011, 2011),
                       months=slice(1,1))
```

To prepare a cutout, the following must be specified for `geodata.Cutout()`:

* The cutout name
* The source dataset
* Time range
* Geographic range as represented by `xy` coordinates.

The example in the code block above uses ERA5 data, as specified by the `module` parameter.

```
module="era5"
```

`xs` and `ys` in combination with the `slice()` function allow us to specify a geographic range based on longitude and latitude.  The above example subsets a portion of Europe.

`years` and `months` are used to subset the time range.  For both functions, the first value represents the start point, and the second value represents the end point.  The above example creates a cutout for January 2011.

## Downloading Data and Creating a Cutout

To download data and create a cutout, run:
```
cutout.prepare();
```
The **geodata** package will create a folder in the cutout directory (`cutout_dir`) you specified in `config.py` with the name specified in `geodata.Cutout()` (in the above example, `merra2-europe-sub24-2011-01`).  The folder, depending on the date range, will then contain one or more monthly netcdf files containing ERA5 data corresponding to the temporal and geographical ranges indicated when the cutout was created.  Data files in the cutout folder will be at the monthly level - i.e., there will be one file for each month in the specified download time range.


To actually create the cutout, you must run `cutout.prepare()`.  Upon running `cutout.prepare()`, **geodata** will check for the presence of the cutout and abort if the cutout already exists.  If you want to force the regeneration of a cutout, run the command with the parameter `overwrite=True`.



## Cutout Metadata

You can query various metadata associated with a cutout.

```
cutout
```
returns the name, geographic and time range, and the preparation status of the cutout (i.e., whether cutout.prepare() has been run, creating the .nc files making up the cutout data).

```
<Cutout era5-europe-test-2011-01 x=30.00-41.50 y=34.81-33.81 time=2011/1-2011/1 prepared>
```

`cutout.name` returns just the name:

```
'era5-europe-test-2011-01'
```

`cutout.coords` returns coordinates:
```
Coordinates:
  * x           (x) float32 30.0 30.25 30.5 30.75 31.0 ... 40.75 41.0 41.25 41.5
  * y           (y) float32 34.815 34.565 34.315 34.065 33.815
  * time        (time) datetime64[ns] 2011-01-01 ... 2011-01-31T23:00:00
    lon         (x) float32 ...
    lat         (y) float32 ...
  * year-month  (year-month) MultiIndex
  - year        (year-month) int64 2011
  - month       (year-month) int64 1
```

`cutout.meta` returns all associated metadata:

```
<xarray.Dataset>
Dimensions:     (time: 744, x: 47, y: 5, year-month: 1)
Coordinates:
  * x           (x) float32 30.0 30.25 30.5 30.75 31.0 ... 40.75 41.0 41.25 41.5
  * y           (y) float32 34.815 34.565 34.315 34.065 33.815
  * time        (time) datetime64[ns] 2011-01-01 ... 2011-01-31T23:00:00
    lon         (x) float32 ...
    lat         (y) float32 ...
  * year-month  (year-month) MultiIndex
  - year        (year-month) int64 2011
  - month       (year-month) int64 1
Data variables:
    height      (y, x) float32 ...
Attributes:
    Conventions:  CF-1.6
    history:      2020-03-14 04:29:35 GMT by grib_to_netcdf-2.16.0: /opt/ecmw...
    module:       era5
    view:         {'x': slice(30, 41.56244222, None), 'y': slice(35, 33.56459...
```


