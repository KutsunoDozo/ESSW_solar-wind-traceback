##Extremely Slow Solar Wind (ESSW) traceback: Mapping L1 in-situ observations to 2.5 R_s
Traces in-situ ESSW events from WIND spacecraft measurements at the L1 point to their solar source regions at 2.5 R_S using a constant-velocity traceback method together with the PFSS (Potential Field Source Surface) model built from SDO/HMI magnetograms.
# Solar Wind Traceback: Mapping WIND Observations to Solar Source Regions

Traces in-situ solar wind measurements from the **WIND** spacecraft at L1 back to their source locations on the Sun, using a ballistic traceback combined with a **PFSS (Potential Field Source Surface)** model built from **SDO/HMI** synoptic magnetograms. The notebook is set up with one example event, an Earth-directed solar wind (ESSW) event on 19-20 Feb 2024 in **Carrington Rotations 2280 and 2281**, but the dates and inputs can be changed to trace any event (see [Tracing a different event](#tracing-a-different-event)).

## Python Dependencies

The notebook was developed and run on Google Colab with the versions below.

1. Python 3.12
2. numpy
3. pandas
4. matplotlib
5. astropy >= 7.2.0
6. sunpy >= 7.1.2
7. astrospice >= 0.2.1
8. pfsspy >= 1.2.0
9. sunkit-magex >= 1.1.0
10. mpl-animators >= 1.2.0
11. reproject >= 0.12.0
12. notebook (Jupyter)

## Data

1. WIND/SWE H1 solar wind measurements (2-minute cadence): available online at https://cdaweb.gsfc.nasa.gov/ (dataset `WI_H1_SWE`).
2. SDO/HMI synoptic Carrington maps (`hmi.synoptic_mr_720s`): available online at http://jsoc.stanford.edu/.

* Note: the notebook expects `WI_H1_SWE_2018_2024.csv` (with a `time` column, `bulk_speed`, and `XGSE_earth_radii`, `YGSE_earth_radii`, `ZGSE_earth_radii`), which it reads from a zip file on Google Drive (`WI_H1_SWE_2018_2024.zip`). The HMI map for each rotation, e.g. `hmi.synoptic_mr_720s.2280.synopMr.fits`, must be placed alongside the notebook. Neither data file is included in this repository.

## Code Structure

The workflow roadmap, all within `traceback6_v2_19Feb2024.ipynb`:

data loading → Carrington Rotation selection → coordinate transformation → ballistic traceback → PFSS modelling → field line tracing → plotting

## Main modules

1. **Data loading** - mounts Google Drive, unzips the WIND CSV, and installs the required packages.
2. **CR selection** - filters WIND data to one Carrington Rotation window and saves it to `WIND_H1_2MIN_<start>_<end>.csv`. CR 2281 is 2024-02-14 to 2024-03-12 and CR 2280 is 2024-01-18 to 2024-02-14.
3. **Coordinate transformation** - converts spacecraft position from GSE to Heliocentric Inertial (HCI) to Heliographic Carrington using `sunpy` and `astropy`.
4. **Ballistic traceback** - computes propagation time, longitudinal offset and footpoint longitude/latitude at the source surface, and saves `wind_traceback6.csv` (CR 2281) and `wind_traceback6(a).csv` (CR 2280).
5. **Carrington longitude lookup** - builds a longitude-vs-timestamp table (360 degrees over 27.2753 days) to find the dates matching a given longitude range.
6. **PFSS modelling** - loads the HMI map, converts it to a CEA projection, resamples it to 360 x 180, cleans NaNs, and runs `pfsspy.pfss` with `nrho = 35` and `rss = 2.5`.
7. **`pfss2flines()`** - helper function that traces field lines on a regular grid, either from the photosphere or from the source surface, and returns the output used for the coronal hole map.
8. **`source_surface_seeds()`** - helper function that builds a 92 x 41 grid of seed points on the source surface; field lines traced from it give the footpoints plotted on the HMI map.
9. **Plotting** - produces speed vs. footpoint longitude, the HMI and source-surface Br maps with polarity inversion lines, the coronal hole map, and a combined figure of speed vs. longitude over the HMI map with polarity-coloured footpoints.

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
- `nrho = 35` radial grid points, source surface at `rss = 2.5 R☉`
- Field lines traced with `pfsspy`'s `FortranTracer`: a 181 x 361 photospheric grid for the coronal hole map, and a 92 x 41 source-surface grid for footpoint plotting
- Open-field polarity coloured red (+), blue (−) and black (closed/undefined)

## Usage

1. Clone the repository and open `traceback6_v2_19Feb2024.ipynb` in Jupyter or Colab.
2. Update the file paths in the first cells. The notebook currently mounts Google Drive and reads from `/content/drive/MyDrive/WI_H1_SWE_2018_2024.zip`.
3. Run the notebook top to bottom. As provided, it traces the Feb 2024 example event.

## Tracing a different event

Only one event is traced in the notebook, but nothing in the method is specific to it. To trace another event, change the following:

1. **Date range:** set `start_date` and `end_date` to the Carrington Rotation (or any window) containing your event, e.g.

   ```python
   # CR 2281
   start_date = '2024-02-14'
   end_date   = '2024-03-12'
   ```

   The WIND CSV must cover these dates.
2. **HMI map:** load the synoptic map for the same rotation, e.g. `hmi.synoptic_mr_720s.2281.synopMr.fits`, and update the rotation number in the plot title.
3. **Carrington longitude lookup:** update `start_datetime` in the longitude-vs-time table to the start time of your rotation.
4. **Event markers:** change the red dashed longitude lines (`axvline`, currently `x=1` and `x=14`) to the footpoint longitude range of your event. Read this off the lookup table or the `footpoint_lon_deg` column for your event's start and end times.
5. **Labels and filenames:** edit the figure label (currently "ESSW from 19 to 20 Feb 2024 CR 2280") and the output CSV names.

## Notes and limitations

- The ballistic approximation assumes constant radial solar wind speed and ignores stream interactions, acceleration, and non-radial flow.
- Carrington rotation start times for the longitude lookup tables are entered manually.
- HMI maps are downsampled to 360 x 180 for speed, and NaN values are set to zero before running PFSS.
- The notebook repeats the same steps for CR 2280 and CR 2281; these could be refactored into a function parameterised by rotation number.

## Acknowledgements

WIND/SWE data courtesy of NASA/GSFC and CDAWeb. HMI data courtesy of NASA/SDO and the HMI science team via JSOC.

## License

Add a license of your choice (e.g. MIT) before publishing.

## Author

*Your name and contact / affiliation here.*
