# Astronomical Image Processing and Stellar Mapping

Analysis of the open star cluster **Berkeley 59** using astronomical image processing, Gaia DR3 astrometric data, and stellar population analysis.

This project was carried out as part of the **Astronomy Club, Institute Technical Council, IIT Bombay**, where raw astronomical observations of Berkeley 59 were processed and combined with publicly available Gaia data to identify cluster members and study their stellar properties.

---

## Overview

The project follows an observational astronomy workflow:

```text
Raw Astronomical Image
        │
        ▼
FITS Image Processing
        │
        ▼
WCS Coordinate Mapping
        │
        ▼
Gaia DR3 Data Retrieval
        │
        ▼
Astrometric Selection
        │
        ▼
Cluster Member Identification
        │
        ├───────────────┐
        ▼               ▼
Member Positions    Photometric Data
        │               │
        │               ▼
        │        Color-Magnitude Diagram
        │               │
        ▼               ▼
   Stellar Mapping ──► Stellar Population Analysis
```

The analysis combines **image data**, **World Coordinate System (WCS) information**, and **Gaia DR3 astrometric and photometric measurements** to connect observed stars with catalogued sources.

---

## Objectives

The main objectives of the project were:

* Process and visualize astronomical FITS images of Berkeley 59.
* Work with observations obtained using different photometric filters.
* Retrieve stellar data from the **Gaia DR3** archive.
* Select reliable stellar sources using astrometric quality criteria.
* Identify likely members of the star cluster using astrometric properties.
* Map Gaia sources back onto the observed astronomical image.
* Construct color-magnitude and Hertzsprung–Russell diagrams.
* Study the stellar population and evolutionary state of the cluster.

---

## Dataset

### Astronomical Image

The project was provided with processed astronomical images of Berkeley 59 in **FITS** format.

The FITS files contain:

* Pixel intensity information
* Observation metadata
* Photometric filter information
* WCS information for converting between pixel and sky coordinates

Example:

```python
from astropy.io import fits

hdul = fits.open('Berk_i.wcs.proc (1).fits')

image_data = hdul[0].data
header = hdul[0].header

print(f"Filter Used: {header.get('FILTER', 'N/A')}")

hdul.close()
```

---

## 1. FITS Image Visualization

The astronomical image was first loaded and visualized using `Astropy`.

Because astronomical images can contain a large dynamic range of pixel intensities, a **ZScale normalization** was used to improve the visualization of faint and bright sources.

```python
from astropy.visualization import ZScaleInterval, ImageNormalize

norm = ImageNormalize(
    image_data,
    interval=ZScaleInterval()
)

plt.imshow(
    image_data,
    cmap='viridis',
    norm=norm,
    origin='lower'
)
```

The resulting image allows individual stellar sources and the overall structure of the observed field to be inspected.

---

## 2. Gaia DR3 Data Retrieval

To obtain accurate astrometric and photometric information for the observed stars, the **Gaia DR3 archive** was queried using `astroquery`.

The query retrieved parameters including:

* Right Ascension (RA)
* Declination (Dec)
* Parallax
* Proper motion in RA
* Proper motion in Dec
* Gaia G-band magnitude
* BP − RP color
* RUWE

Example query:

```sql
SELECT 
    source_id,
    ra,
    dec,
    parallax,
    pmra,
    pmdec,
    phot_g_mean_mag,
    bp_rp,
    ruwe
FROM gaiadr3.gaia_source
WHERE 
    CONTAINS(
        POINT('ICRS', ra, dec),
        CIRCLE('ICRS', 0.5, 67.4, 0.8)
    ) = 1
    AND parallax > 0
    AND parallax_over_error > 5
    AND ruwe < 1.4
    AND phot_g_mean_mag < 19
```

The initial selection was performed within a specified angular region around the cluster.

---

## 3. Astrometric Quality Filtering

Gaia sources were filtered using astrometric quality criteria to reduce unreliable measurements.

The selection included:

```text
Parallax > 0
Parallax / Parallax Error > 5
RUWE < 1.4
G magnitude < 19
```

These criteria help restrict the analysis to sources with sufficiently reliable astrometric measurements.

The resulting Gaia catalog was stored locally:

```python
df_gaia.to_csv('gaia_results.csv', index=False)
```

---

## 4. Cluster Member Identification

Cluster membership was investigated using Gaia astrometric information, particularly the proper-motion and parallax measurements.

The objective was to distinguish stars physically associated with Berkeley 59 from unrelated foreground and background stars.

The resulting member catalog was then used for subsequent image mapping and photometric analysis.

---

## 5. Connecting Gaia Coordinates to the Image

The FITS image contains WCS information that allows celestial coordinates to be converted into image pixel coordinates.

For a Gaia source with:

$$
(\alpha,\delta) = (\text{RA},\text{Dec}),
$$

the WCS transformation gives:

$$
(\alpha,\delta)
\rightarrow
(x,y).
$$

This was implemented using `Astropy WCS`:

```python
from astropy.wcs import WCS

wcs = WCS(hdul[0].header)

world_coords = members_df[['ra', 'dec']].values

pixel_x, pixel_y = wcs.wcs_world2pix(
    world_coords[:, 0],
    world_coords[:, 1],
    0
)
```

Only sources falling within the image boundaries were retained.

```python
mask = (
    (pixel_x >= 0) &
    (pixel_x < image_data.shape[1]) &
    (pixel_y >= 0) &
    (pixel_y < image_data.shape[0])
)
```

---

## 6. Stellar Mapping

The identified Gaia cluster members were overlaid on the astronomical image.

This provides a visual check connecting the catalogued stellar sources to their corresponding locations in the observed field.

```python
plt.imshow(
    image_data,
    cmap='gray',
    norm=norm,
    origin='lower'
)

plt.scatter(
    filtered_x,
    filtered_y,
    s=80,
    edgecolors='red',
    facecolors='none',
    lw=2,
    label='Confirmed Cluster Members'
)
```

The resulting map highlights the spatial distribution of the selected cluster members directly on the astronomical image.

---

## 7. Color-Magnitude Diagram

The photometric data were used to construct a **Color-Magnitude Diagram (CMD)**.

For each star, the color index was calculated from photometric measurements:

$$
\text{Color} = g-r.
$$

The magnitude was plotted on the vertical axis.

```python
plt.scatter(
    members_in_frame['g_r'],
    members_in_frame['mag_g'],
    s=15,
    alpha=0.7
)

plt.gca().invert_yaxis()

plt.xlabel('Color (g - r)')
plt.ylabel('Magnitude (g)')
```

The magnitude axis is inverted because **smaller astronomical magnitudes correspond to brighter stars**.

The resulting CMD can be used to investigate stellar population features such as the main sequence and other evolutionary structures.

---

## 8. Hertzsprung–Russell Diagram

The stellar photometric data were also used to study the distribution of stars in color–magnitude space.

The CMD serves as an observational representation of the **Hertzsprung–Russell diagram**, allowing the stellar population to be examined in terms of brightness and color.

Features of interest include:

* Main-sequence population
* Evolved stars
* Turn-off region
* Distribution of stellar colors
* Relative stellar populations

These features provide information about the evolutionary state of the cluster.

---

## 9. Isochrone Analysis

The observed stellar distribution can be compared with theoretical **isochrones** corresponding to different stellar ages.

By examining the agreement between the observed CMD and theoretical stellar-evolution tracks, the approximate age and evolutionary state of the cluster can be investigated.

The analysis can be extended by comparing:

```text
Observed CMD
      │
      ▼
Stellar Evolution Models
      │
      ▼
Isochrone Comparison
      │
      ▼
Cluster Age / Evolutionary State
```

---

## 10. Technologies Used

### Python

The analysis was performed primarily in Python.

### Libraries

* **Astropy** — FITS files, WCS transformations, astronomical visualization
* **Astroquery** — Gaia archive queries
* **NumPy** — numerical computation
* **Pandas** — tabular data processing
* **Matplotlib** — image visualization and astronomical plots

---

## 11. Repository Structure

```text
Astronomical-Image-Processing/
│
├── README.md
│
├── images/
│   └── Berkeley59/
│
├── data/
│   ├── gaia_results.csv
│   └── photometry/
│
├── notebooks/
│   ├── image_processing.ipynb
│   ├── gaia_analysis.ipynb
│   └── stellar_mapping.ipynb
│
├── plots/
│   ├── cluster_image.png
│   ├── member_mapping.png
│   └── color_magnitude_diagram.png
│
└── requirements.txt
```

---

## 12. Key Concepts

This project provided practical experience with:

* Astronomical image processing
* FITS data
* Astropy
* World Coordinate Systems (WCS)
* Gaia DR3
* Astrometric data analysis
* Proper motion
* Parallax
* Photometric measurements
* Stellar cluster membership
* Color-Magnitude Diagrams
* Hertzsprung–Russell diagrams
* Isochrone analysis
* Coordinate transformations
* Scientific data visualization

---

## Results

The analysis produced:

1. Processed visualizations of the Berkeley 59 observation field.
2. A filtered Gaia DR3 stellar catalog.
3. Astrometrically selected candidate cluster members.
4. A spatial map of identified members overlaid on the astronomical image.
5. A color-magnitude diagram for studying the stellar population.
6. An isochrone-based approach for estimating the evolutionary state of the cluster.

### Example: Cluster Member Mapping

*Add the final member-overlay image here.*

### Example: Color-Magnitude Diagram

*Add the final CMD here.*

---

## References

* **Gaia Data Release 3 (DR3)** — European Space Agency Gaia mission
* **Astropy** — Python ecosystem for astronomy
* Gaia Archive / `astroquery.gaia`
* Stellar evolution and isochrone models used during the analysis

---

## Project Context

**Astronomy Club, Institute Technical Council, IIT Bombay**

**Duration:** June 2024 – August 2024
