Data

This project uses two HDF5 observation products for GJ 144 obtained from the Breakthrough Listen Open Data Archive.

The raw .h5 files are not included in this repository because of their size. They can be downloaded directly from the original archive using the links below.

File 1 — Time-Resolved Observation

Filename:
spliced_blc0001020304050607_guppi_57635_35657_Gj144_0005.gpuspec.0002.h5

Target: GJ 144
Format: HDF5 (.h5)
File size: ~263.7 MB
Frequency coverage: 1797.952–2802.832 MHz
Frequency resolution: ~2.861 kHz
Time resolution: ~1.074 s
Time integrations: 273
Observation start: 2016-09-04 09:54:17 UTC
Coordinates: RA 03:32:54.68, Dec −09:13:11.007

Download:
Breakthrough Listen Open Data Archive — .0002.h5

Use in this project

This file provides relatively fine time resolution and is used for the initial exploration and time-frequency characterization of candidate structure.

The initial exploratory analysis examines a 2200–2205 MHz subset of the full observation before narrowing the analysis to smaller candidate regions.

⸻

File 2 — High-Frequency-Resolution Observation

Filename:
spliced_blc0001020304050607_guppi_57635_35657_Gj144_0005.gpuspec.0000.h5

Target: GJ 144
Format: HDF5 (.h5)
File size: ~17.33 GB
Frequency coverage: 1797.949–2802.832 MHz
Frequency resolution: ~2.794 Hz
Time resolution: ~18.254 s
Time integrations: 16
Observation start: 2016-09-04 09:54:17 UTC
Coordinates: RA 03:32:54.68, Dec −09:13:11.007

Download:
Breakthrough Listen Open Data Archive — .0000.h5

Use in this project

This file provides substantially finer frequency resolution at the expense of time resolution. It is used for higher-resolution spectral inspection of candidate structure identified during the earlier exploratory analysis.

The higher spectral resolution enables examination of fine-scale features that are unresolved or blended together in the lower-frequency-resolution observation.

⸻

Resolution Tradeoff

The two files represent different time-frequency resolutions of the same GJ 144 observation.

Property	.0002.h5	.0000.h5
File size	~263.7 MB	~17.33 GB
Frequency resolution	~2.861 kHz	~2.794 Hz
Time resolution	~1.074 s	~18.254 s
Time integrations	273	16
Frequency channels	351,232	359,661,568

The .0002 product provides substantially finer temporal sampling and is useful for examining how candidate behavior changes through time. The .0000 product provides much finer spectral sampling and is useful for resolving fine frequency structure.

Using both products allows candidate behavior to be examined at complementary time and frequency resolutions.

Data Availability

The raw observation files are intentionally excluded from version control because of their size. To reproduce the analysis, download the appropriate HDF5 files from the Breakthrough Listen Open Data Archive using the links above and place them in the project’s local data/ directory.
