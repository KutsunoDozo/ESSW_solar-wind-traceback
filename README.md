## Extremely Slow Solar Wind (ESSW) traceback: Mapping L1 in-situ observations to 2.5 R_s
Traces in-situ ESSW events from WIND spacecraft measurements at the L1 point to their solar source regions at 2.5 R_S using a constant-velocity traceback method together with the PFSS (Potential Field Source Surface) model built from SDO/HMI synoptic magnetograms. A second notebook renders the PFSS field lines and the magnetic neutral line in a 3D (orthographic) view over HMI full-disk magnetograms. The notebook is set up with one example event on 17-19 June 2013 in Carrington Rotation 2138, but the dates and inputs can be changed to trace any event. 

## Python Dependencies

The notebooks were developed and run on Google Colab with the versions below.

1. Python 3.12
2. numpy
3. pandas
4. matplotlib
5. astropy >= 7.2.0
6. sunpy >= 7.1.2 
7. astrospice >= 0.2.1
8. pfsspy >= 1.2.0
9. sunkit-magex >= 1.1.0
10. streamtracer >= 2.6.0
11. drms
12. mpl-animators >= 1.2.0
13. reproject >= 0.12.0

## Data

1. WIND/SWE H1 solar wind measurements: available online at https://cdaweb.gsfc.nasa.gov/.
2. SDO/HMI synoptic Carrington maps: available online at http://jsoc.stanford.edu/.
3. SDO/HMI full-disk line-of-sight magnetograms: available online at http://jsoc.stanford.edu/.


## Code Structure

`traceback20_v2.ipynb`: data loading → Carrington Rotation selection → coordinate transformation → ballistic traceback → PFSS modelling → field line tracing → plotting

`pfss_3d_v1.ipynb`: JSOC download → PFSS modelling → field line tracing (closed and open) → orthographic 3D plots over HMI magnetograms

## Main modules

### `traceback20_v2.ipynb`

1. **Data loading** - installs the packages, mounts Google Drive, and reads the WIND CSV.
2. **CR selection** - filters WIND data to one Carrington Rotation window and saves it to `WIND_H1_5MIN_<start>_<end>.csv`. The example is CR 2138 (2013-06-11 to 2013-07-08).
3. **Coordinate transformation** - converts spacecraft position from GSE to Heliocentric Inertial (HCI) to Heliographic Carrington using `sunpy` and `astropy`.
4. **Ballistic traceback** - computes propagation time, longitudinal offset and footpoint longitude/latitude at the source surface, and saves `wind_traceback20.csv`.
5. **Carrington longitude lookup** - builds a longitude-vs-timestamp table (360 degrees over 27.2753 days) to find the dates matching a given longitude range.
6. **Speed filter** - removes bad speed values (keeps 0 to 2000 km/s) before plotting.
7. **PFSS modelling** - loads the HMI map, converts it to a CEA projection, resamples it to 360 x 180, cleans NaNs, and runs `pfsspy.pfss` with `nrho = 35` and `rss = 2.5`.
8. **`pfss2flines()`** - helper function that traces field lines on a regular grid, from the photosphere or from the source surface, and returns the output used for the coronal hole map.
9. **`source_surface_seeds()`** - helper function that builds a 92 x 41 grid of seed points on the source surface; field lines traced from it give the footpoints plotted on the HMI map.
10. **Plotting** - speed vs. footpoint longitude, HMI and source-surface Br maps with polarity inversion lines, the coronal hole map, and a combined figure of speed vs. longitude over the HMI map with polarity-coloured footpoints.

### `pfss_3d_v1.ipynb`

1. **JSOC download** - searches and fetches the HMI synoptic map (`hmi.synoptic_mr_polfil_720s`, CR 2138) with `Fido`.
2. **PFSS modelling** - same pipeline as above, with a finer radial grid (`nrho = 60`, `rss = 2.5`).
3. **`pfss2flines()`** - traces closed field lines (from the photosphere) and open field lines (from the source surface) on a 36 x 72 grid.
4. **`sph2orthographic()`** - projects (r, lon, lat) points onto the plane of the sky as seen from a chosen central meridian longitude (CMP) and the solar B0 angle, keeping only visible points.
5. **`plot_pfss_orthographic()`** - draws a lat/lon grid, the magnetic neutral line, and the field lines coloured by polarity, with circles for the photosphere and the source surface.
6. **`plot_pfss_orthographic_with_hmi()`** - the same view with an HMI full-disk magnetogram (`hmi.m_45s`) as the photospheric background. The example makes one plot at each end of the event, at CMP 351° and 331°.

## Method

### Ballistic traceback

For each WIND measurement, the travel time from the source surface to the spacecraft is

```
Δt = (r_spacecraft − R_ss) / v_sw
```

where `R_ss = 2.5 R☉` and `v_sw` is the measured bulk speed. During that time the Sun rotates, so the longitudinal offset is

```
D = Ω · Δt,   Ω = 360° / (27.2753 days)
```

and the footpoint longitude at the source surface is

```
φ_footpoint = (φ_Carrington + D) mod 360°
```

The footpoint latitude is taken as the spacecraft's Carrington latitude.

### PFSS modelling

- HMI synoptic map converted to a cylindrical equal-area (CEA) projection and resampled to 360 x 180 pixels
- Source surface at `rss = 2.5 R☉`, with `nrho = 35` radial grid points in the traceback notebook and `nrho = 60` in the 3D notebook
- Field lines traced with `pfsspy`'s `FortranTracer`
- Open-field polarity coloured red (+) and blue (−), with closed or undefined lines in black (traceback notebook) or green (3D notebook)

## Usage

1. Clone the repository and open the notebooks in Jupyter or Colab.
2. Update the file paths in `traceback20_v2.ipynb`. It currently reads from `/content/drive/MyDrive/`.
3. In `pfss_3d_v1.ipynb`, replace the JSOC notification email with your own registered address.
4. Run the notebooks top to bottom. As provided, they trace the June 2013 example event.

## Tracing a different event

Only one event is traced in the notebooks, but nothing in the method is specific to it. To trace another event, change the following.

In `traceback20_v2.ipynb`:

1. **Input data and column names:** point to a WIND CSV covering your dates. If your file uses different column names, update the time and speed column names throughout.
2. **Date range:** set `start_date` and `end_date` to the Carrington Rotation (or any window) containing your event, e.g.

   ```python
   # CR 2138
   start_date = '2013-06-11'
   end_date   = '2013-07-08'
   ```

3. **HMI map:** load the synoptic map for the same rotation, e.g. `hmi.synoptic_mr_720s.2138.synopMr.fits`, and update the rotation number in the plot title.
4. **Carrington longitude lookup:** update `start_datetime` in the longitude-vs-time table to the start time of your rotation.
5. **Event markers:** change the red dashed longitude lines (`axvline`, currently `x=331` and `x=351`) to the footpoint longitude range of your event. Read this off the lookup table or the `footpoint_lon_deg` column for your event's start and end times.
6. **Labels and filenames:** edit the figure label (currently "ESSW from 17 to 19 June 2013 CR 2138") and the output CSV names.

In `pfss_3d_v1.ipynb`:

1. **Time and rotation:** update the JSOC time range and `CAR_ROT` number (currently 2138) and the loaded filename.
2. **Full-disk magnetograms:** change the `hmi.m_45s` search times to the start and end of your event, and update the downloaded filenames.
3. **Viewing longitude:** set `lon0` in the plotting calls to the CMP longitude for each time (currently 351 and 331).

## Notes and limitations

- The ballistic approximation assumes constant radial solar wind speed and ignores stream interactions, acceleration, and non-radial flow.
- Carrington rotation start times for the longitude lookup tables are entered manually.
- HMI maps are downsampled to 360 x 180 for speed, and NaN values are set to zero before running PFSS in the traceback notebook.
- In `pfss_3d_v1.ipynb`, run cells in order: `flines` is used for plotting before it is created in a later cell, and `astropy` must be imported before `pfss2flines()` is called.

## Acknowledgements

WIND/SWE data courtesy of NASA/GSFC and CDAWeb. HMI data courtesy of NASA/SDO and the HMI science team via JSOC.

## License

Add a license of your choice (e.g. MIT) before publishing.

## Author

*Your name and contact / affiliation here.*


