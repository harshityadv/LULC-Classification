# DAT-103 Indonesia Remote Sensing & Land Cover Analysis

A 5-member team project for the **DAT-103 Remote Sensing** course that builds a complete automated pipeline for Land Use / Land Cover (LULC) analysis over an Area of Interest (AOI) in **Bogor, Indonesia** using Landsat 8/9 imagery.

The pipeline covers everything from raw satellite data acquisition to CNN-based segmentation training.

---

## Project Structure

```
Project1/
├── Notebook1_Landsat_Download.ipynb   # Member 1 – Data Acquisition
├── DATNotebook2 (3).ipynb             # Member 2 – LULC Label Acquisition & Alignment
├── Notebook3_NDVI_(1).ipynb           # Member 3 – NDVI Calculation
├── Notebook4_Data_Pipeline.ipynb      # Member 4 – Data Pipeline & Patch Creation
└── DAT_Segmentation_Training.ipynb    # Member 5 – CNN Segmentation Training
```

### Google Drive Folder Layout (shared between all members)

```
DAT103_Indonesia_Project/
├── data/
│   ├── landsat/      ← Downloaded & clipped Landsat bands (TIF)
│   ├── lulc/         ← Raw LULC rasters
│   ├── processed/    ← NDVI, reprojected LULC, etc.
│   └── patches/      ← Image patches for model training
├── models/           ← Saved model checkpoints
└── figures/          ← Output visualisations
```

---

## Area of Interest (AOI)

| Parameter | Value |
|-----------|-------|
| Location  | Bogor City, West Java, Indonesia |
| Min. Longitude | 107.2831 |
| Max. Longitude | 107.3471 |
| Min. Latitude  | -6.7023  |
| Max. Latitude  | -6.6383  |
| Approx. Area   | ~50 km² |
| CRS | EPSG:32648 (UTM Zone 48S) |

---

## Notebook Descriptions

### 1. `Notebook1_Landsat_Download.ipynb` – Data Acquisition
**Member 1**

Downloads Landsat 8/9 Collection 2 Level-2 surface-reflectance imagery for the AOI via the **USGS M2M API**.

**What it does:**
- Authenticates with the USGS Earth Explorer M2M API
- Searches for and selects a suitable scene (low cloud cover, correct date range)
- Downloads individual spectral bands (B2–B7) as GeoTIFF files
- Clips each band to the AOI bounding box
- Saves metadata JSON alongside the bands

**Key dependencies:**
```
requests, rasterio, tqdm, matplotlib, numpy
```

> **Security Note:** The notebook contains a hardcoded USGS username and API token. Before sharing or committing to a public repository, move these credentials to a `.env` file or use Google Colab's **Secrets** feature and load them with `userdata.get('KEY')`.

---

### 2. `DATNotebook2 (3).ipynb` – LULC Label Acquisition & Alignment
**Member 2**

Downloads ESRI's 10 m global Land Use / Land Cover map (9-class) via the **Microsoft Planetary Computer STAC API** and aligns it to the Landsat grid.

**What it does:**
- Mounts Google Drive and sets up shared folder paths
- Queries the `io-lulc-9-class` STAC collection for the AOI
- Clips the downloaded LULC raster to the AOI geometry
- Reprojects / resamples the 10 m ESRI LULC to match the 30 m Landsat grid (nearest-neighbour)
- Saves the aligned raster to `data/processed/lulc_reprojected.tif`
- Produces a colour-coded LULC map figure saved to `figures/lulc_map.png`

**ESRI LULC Classes:**

| ID | Class | Hex Color |
|----|-------|-----------|
| 1  | Water | `#1A5BAB` |
| 2  | Trees | `#358221` |
| 4  | Flooded Vegetation | `#87D19E` |
| 5  | Crops | `#FFDB5C` |
| 7  | Built Area | `#ED022A` |
| 8  | Bare Ground | `#EDE9E4` |
| 11 | Rangeland | `#C6AD8D` |

**Key dependencies:**
```
rasterio, geopandas, shapely, planetary-computer, pystac-client, numpy, matplotlib
```

---

### 3. `Notebook3_NDVI_(1).ipynb` – NDVI Calculation
**Member 3**

Computes the Normalized Difference Vegetation Index (NDVI) from the downloaded Landsat bands.

**What it does:**
- Loads Landsat Band 4 (Red) and Band 5 (NIR) from the shared Drive
- Applies the Landsat Collection 2 Level-2 scale factors:  
  `reflectance = raw_value × 0.0000275 − 0.2`
- Masks invalid pixels (nodata and saturation values)
- Computes NDVI: `(NIR − Red) / (NIR + Red)`
- Clips result to the range `[−1, 1]`
- Saves `data/processed/ndvi.tif` (Float32, nodata = −9999)
- Outputs a colour-bar NDVI map to `figures/`

**Sample statistics from a run over Bogor:**
- NDVI range: `[−0.031, 1.000]`
- Mean NDVI: `0.623`
- Pixels > 0.5 (dense vegetation): **79.6 %**
- Pixels < 0.1 (bare/water): **0.9 %**

**Key dependencies:**
```
rasterio, numpy, matplotlib
```

---

### 4. `Notebook4_Data_Pipeline.ipynb` – Data Preparation & Patch Creation
**Member 4**

Assembles all processed rasters into stacked multi-band images and cuts them into fixed-size patches ready for deep-learning training.

**What it does:**
- Stacks Landsat bands + NDVI into a single multi-channel array
- Pairs each image patch with the corresponding LULC label patch
- Applies data augmentation (flips, rotations, etc.)
- Saves patch arrays to `data/patches/`

**Key dependencies:**
```
rasterio, numpy, scikit-learn (or custom utilities)
```

---

### 5. `DAT_Segmentation_Training.ipynb` – CNN Segmentation Training
**Member 5**

Trains a Convolutional Neural Network (CNN) for pixel-wise land-cover classification on the prepared patches.

**What it does:**
- Loads patch dataset from `data/patches/`
- Defines and compiles a segmentation CNN (e.g. U-Net style)
- Trains with GPU acceleration (Google Colab T4)
- Evaluates on a held-out validation split
- Saves the best model checkpoint to `models/`
- Generates prediction maps and accuracy metrics

**Key dependencies:**
```
tensorflow / keras (or pytorch), numpy, matplotlib, scikit-learn
```

---

## How to Run on Google Colab

> All notebooks are designed to run on **Google Colab** with a shared Google Drive folder.

### Prerequisites
| Requirement | Details |
|------------|---------|
| Google Account | Needed for Google Colab & Drive |
| Google Drive | Shared folder `DAT103_Indonesia_Project/` must exist |
| USGS EarthExplorer Account | Required for Notebook 1 only |
| Microsoft Planetary Computer | No account needed (public API) |
| GPU runtime | Recommended for Notebook 5 (Colab → Runtime → Change runtime type → T4 GPU) |

### Step-by-step

1. **Upload notebooks** to Google Colab (or open from Drive).
2. **Configure the shared folder path** in every notebook:
   ```python
   # Option A – personal Drive (default in Notebook 1)
   BASE = '/content/drive/MyDrive/DAT103_Indonesia_Project'

   # Option B – shared Drive (used in Notebooks 2-5)
   SHARED_FOLDER_ID = '1T7uRYN0Ek6H-HEdjIqgXVq_FarJ_hrJk'  # ← update with your ID
   BASE = f'/content/drive/.shortcut-targets-by-id/{SHARED_FOLDER_ID}/DAT103_Indonesia_Project'
   ```
3. **Run Notebook 1** to download Landsat imagery.
4. **Run Notebook 2** to download and align the LULC labels.
5. **Run Notebook 3** to compute the NDVI raster.
6. **Run Notebook 4** to build the patch dataset.
7. **Run Notebook 5** to train and evaluate the segmentation model.

Each notebook installs its own dependencies via `%pip install …` cells — no manual setup required.

---

## Full Dependency List

| Package | Used in |
|---------|---------|
| `rasterio` | All notebooks |
| `numpy` | All notebooks |
| `matplotlib` | All notebooks |
| `requests` | Notebook 1 |
| `tqdm` | Notebook 1 |
| `geopandas` | Notebook 2 |
| `shapely` | Notebook 2 |
| `planetary-computer` | Notebook 2 |
| `pystac-client` | Notebook 2 |
| `tensorflow` / `torch` | Notebook 5 |
| `scikit-learn` | Notebooks 4, 5 |

---

## Known Issues & Notes

- **Credentials in Notebook 1:** A USGS username and API token are currently hardcoded. Replace them with environment variables or Colab Secrets before making the repository public.
- **Shared Drive ID:** The `SHARED_FOLDER_ID` in Notebooks 2–5 must be updated to match your own shared Drive folder.
- **Execution order matters:** Notebooks must be run in sequence (1 → 2 → 3 → 4 → 5). Running them out of order will cause file-not-found errors.
- **GPU for Notebook 5:** Training will be very slow on CPU. Enable a GPU runtime in Colab.

---

## Team

| Member | Notebook | Task |
|--------|----------|------|
| 1 | `Notebook1_Landsat_Download.ipynb` | Satellite data acquisition via USGS M2M API |
| 2 | `DATNotebook2 (3).ipynb` | LULC ground-truth acquisition & spatial alignment |
| 3 | `Notebook3_NDVI_(1).ipynb` | NDVI computation & vegetation analysis |
| 4 | `Notebook4_Data_Pipeline.ipynb` | Data pipeline & training patch creation |
| 5 | `DAT_Segmentation_Training.ipynb` | CNN segmentation model training & evaluation |

---

## License

This project was created for educational purposes as part of the **DAT-103** course. All satellite imagery is sourced from publicly available datasets (Landsat via USGS, LULC via ESRI/Microsoft Planetary Computer).
