# H5 Narrowband Candidate Analysis

Exploratory time-frequency analysis and characterization of narrowband radio
candidates.

## Project Motivation

The goal of this project is to examine time-frequency radio data from an
observation of GJ 144 to identify anomalous narrowband candidates. Candidate
signals are then characterized by examining their temporal and spectral
behavior, including how their structure changes across time and frequency.

The project was motivated by an interest in learning how narrowband signals
can be identified and characterized in real radio-telescope time-frequency
data. More broadly, it serves as a practical introduction to the
signal-analysis methods used in SETI and as a foundation for future work
investigating potential technosignature candidates.

## Analysis

### Phase 1 — Observation Exploration and Candidate Detection
...

The first stage of the project focused on understanding the structure of the
observation before attempting candidate detection. I inspected the dimensions
of the HDF5 data, identified the frequency, feed, and time axes, determined the
number of frequency channels and time samples, calculated the spacing between
samples, and established the total time and frequency coverage of the
observation.

After establishing the structure of the observation, I calculated the mean
power of each frequency bin across time, producing a time-averaged spectrum.
I used `argmax` to identify the frequency bin with the maximum mean power and
progressively visualized narrower frequency windows around this region to
inspect its spectral structure.

I then calculated z-scores for the frequency bins and applied a 3σ threshold,
selecting bins whose mean power differed from the reference mean by at least
three standard deviations in either direction. These threshold crossings were
treated as candidate frequencies for further analysis rather than as confirmed
anomalous signals.

To group related threshold crossings into spectral features, I compared the
separation between consecutive detections with the channel spacing reported in
the observation header. A grouping tolerance of 1.5 times the native channel
spacing (~4.29 kHz) was used to identify discontinuities between detections.
Above-threshold bins separated by more than this tolerance were assigned to
different candidate groups.

The resulting groups were converted into structured candidate records
containing channel count, center frequency, frequency span, strongest z-score,
and signed strongest z-score. This produced a candidate table for subsequent
filtering and time-frequency characterization.


### Phase 2 — Reusable Candidate Detection Pipeline

The second phase of the project focused on converting the exploratory
candidate-detection logic developed in Phase 1 into a reusable analysis
pipeline. The goal was to replace one-off analysis steps with functions that
could be applied consistently to additional frequency windows and observations.

The main `process_observation()` function accepts an observation, a frequency
window size, a z-score detection threshold, and a grouping tolerance. It
divides the observation into frequency windows, processes each window through
the candidate-detection pipeline, and returns a combined list of structured
candidate records.

The windowed approach allows candidate significance to be evaluated against a
local rather than observation-wide spectral background. Within each frequency
window, mean spectral power is extracted and z-scores are calculated using the
power distribution of that window. This allows frequency bins to be identified
as statistically unusual relative to their local spectral environment.

For each window, the pipeline applies the selected z-score threshold, groups
nearby above-threshold detections according to the channel-spacing tolerance,
and summarizes the resulting spectral features into candidate records. The
records from all windows are then combined into a single candidate list for
subsequent analysis.

### Phase 3 — Time-Frequency Candidate Characterization

Phase 3 moved beyond candidate detection to examine how identified features
behave across both time and frequency. Candidate regions were analyzed across
individual time samples to characterize their persistence, temporal
continuity, spectral width, and frequency behavior.

#### Temporal Behavior

Candidate persistence was measured as the fraction of the observation during
which the feature exceeded the 3σ detection threshold. Continuous detection
runs were also identified to distinguish overall persistence from uninterrupted
detection.

For the candidate examined in this phase, the feature was detected above the
3σ threshold for 61.9% of the observation across 11 separate detection runs.
The mean continuous run duration was 16.50 seconds, with the longest continuous
detection lasting 89.12 seconds.

These measurements indicate that the candidate was persistent but intermittent:
it was detected for a majority of the observation, but its detections were
divided into multiple separate runs rather than forming one continuous event.

#### Spectral Behavior

The spectral width of the candidate was measured using full width at half
maximum (FWHM) across time. The candidate exhibited a mean FWHM bandwidth of
approximately 29.00 kHz, with a standard deviation of approximately 1.67 kHz
and an observed range of approximately 22.89–31.47 kHz.

These measurements indicate that the feature maintained a relatively stable
spectral width when detected while still exhibiting some variation through
time.

#### Frequency Drift

The peak-frequency location of the candidate was tracked through time to
investigate whether its frequency changed systematically during the
observation. A linear model was fitted to the measured peak-frequency
trajectory using `numpy.polyfit`, with the fitted slope used as an estimate
of frequency drift.

Using the discrete frequency bins directly produced an estimated drift rate
of +22.55 Hz/s with R² = 0.161. The positive slope indicates an overall
increase in measured peak frequency through time, while the low R² indicates
that a single linear model explains only a limited portion of the observed
frequency variation.

I also examined whether changes in the measured peak frequency corresponded
to the discrete frequency resolution of the observation. Because the initial
peak-frequency measurements were restricted to the centers of the available
frequency bins, apparent frequency changes could occur in discrete steps as
the maximum-power location moved between neighboring bins.

To obtain a finer estimate of the peak location, I applied three-point
parabolic interpolation using the peak bin and its two neighboring frequency
bins. The interpolation calculates a fractional-bin offset from the relative
power of the three bins. This offset was multiplied by the native frequency-bin
spacing and added to the discrete peak frequency to produce a sub-bin estimate
of the spectral peak location.

The interpolated peak-frequency trajectory produced an estimated drift rate
of +23.15 Hz/s with R² = 0.253. The increase in R² indicates that the linear
model explains a larger proportion of the variation in the interpolated
frequency estimates than in the discrete-bin measurements. However, the R²
remains relatively low, so the result does not establish a strongly coherent
linear frequency drift.

Sub-bin interpolation provides a more precise estimate of the peak location
within the existing spectral samples; it does not increase the native
frequency resolution of the observation.

Overall, Phase 3 transformed the detected candidate from a static spectral
feature into a quantitatively characterized time-frequency feature. Temporal
persistence, detection runs, spectral bandwidth, peak-frequency evolution,
linear drift estimates, and sub-bin peak estimation provided a more complete
description of the candidate's behavior for subsequent investigation.


### Phase 4 — High-Resolution Candidate Investigation

Phase 4 extended the candidate investigation using a much larger observation
product with substantially higher frequency resolution but lower temporal
resolution. This complementary view of the same GJ 144 observation made it
possible to examine spectral structure that could not be resolved clearly in
the lower-resolution data.

The original observation product had a frequency spacing of approximately
2.861 kHz per channel, while the high-resolution product provided a spacing of
approximately 2.794 Hz per channel. This increase in frequency resolution
allowed much finer structure within the candidate region to be examined,
although the number of available time samples was reduced.

#### Comb-Like Spectral Structure

When the candidate region was visualized at the higher frequency resolution,
a repeating comb-like structure became visible. Multiple narrow spectral peaks
appeared at approximately regular frequency intervals.

The frequency window was progressively widened to determine whether this
structure was confined to the immediate candidate region or extended across a
broader portion of the spectrum.

After identifying the pattern visually, I quantified the spacing between
consecutive high-resolution spectral peaks by calculating the differences
between their detected peak frequencies. The resulting peak separations showed
a characteristic spacing of approximately 500 Hz, supporting the presence of
a repeated comb-like spectral structure.

The presence of regular spectral spacing was treated as a structural property
of the candidate rather than evidence of a particular signal origin.

#### High-Resolution Temporal Analysis

The higher-resolution product was also used to examine how the candidate
behaved through time. Because frequency location can vary between time samples,
I compared fixed-frequency persistence with drift-aware persistence.

Fixed-frequency persistence measures how often the candidate remains detectable
within a stationary frequency region. Drift-aware persistence instead follows
the changing peak-frequency location through time and evaluates detection
relative to that tracked trajectory.

The drift-aware analysis detected the candidate in 8 of 16 time samples,
corresponding to a persistence of 50%. These detections occurred across several
separate runs with lengths of 2, 1, 2, and 3 samples. The longest continuous
tracked detection lasted approximately 54.76 seconds.

The fixed-frequency analysis detected the feature in 7 of 16 time samples,
corresponding to a persistence of 43.75%. Although the overall persistence was
slightly lower, these detections formed a single continuous run lasting
approximately 127.78 seconds.

This difference shows that persistence alone does not fully describe temporal
behavior. The drift-aware method recovered detections across a larger fraction
of the observation, but those detections were more fragmented. In contrast,
the fixed-frequency method detected the feature less often overall but captured
one substantially longer continuous interval.

Together, these results suggest that the candidate's spectral behavior varies
through time rather than remaining consistently detectable along a single
simple trajectory.

#### High-Resolution Frequency Drift

Peak frequency was tracked across the available high-resolution time samples
and fitted with a linear model to estimate frequency drift.

The fitted drift rate was approximately +0.0696 Hz/s. However, the
corresponding R² was approximately 0.057, indicating that the linear model
explained only a small portion of the observed frequency variation.

The fitted slope was therefore retained as a descriptive feature rather than
interpreted as evidence of coherent linear frequency drift. Although the
candidate's peak frequency changes through time, those changes are not well
described by a single linear trajectory.

#### Phase 4 Interpretation

Phase 4 demonstrated the importance of examining the same observation at
different resolutions. The higher-frequency-resolution product revealed a
regular comb-like structure with approximately 500 Hz peak spacing that could
not be resolved individually in the lower-resolution observation.

Temporal characterization further showed that the feature was intermittent and
that its measured persistence depended on whether its frequency location was
treated as fixed or tracked through time. At the same time, the very low R² of
the high-resolution drift fit provided little support for describing the
candidate as a coherently linearly drifting signal.

The candidate therefore remained an interesting structured spectral feature,
but its measurements alone were not sufficient to determine its physical
origin.


### Phase 5 — Machine Learning Feasibility

Phase 5 evaluated whether the candidate features developed in the previous
phases could support a meaningful machine-learning analysis. Rather than
immediately fitting a model, I first examined whether the available data
satisfied the requirements for supervised or unsupervised learning.

#### Supervised Learning

Supervised learning requires observations paired with reliable ground-truth
labels. Although the previous phases produced quantitative features describing
the candidate's temporal and spectral behavior, those measurements alone could
not establish the physical origin of the detected feature.

A statistically unusual or structured signal cannot automatically be labeled
as a technosignature, RFI, or another physical signal class based solely on its
appearance in this observation. Assigning such labels without independent
evidence would create unsupported ground truth and would make subsequent model
performance difficult to interpret.

The available analysis also contained too few independently characterized
candidates to provide meaningful examples of multiple signal classes.
Supervised classification was therefore not considered appropriate for the
current dataset.

#### Unsupervised Learning

Unsupervised learning does not require predefined class labels and could
eventually be useful for identifying clusters, unusual candidates, or recurring
signal morphologies.

However, the current analysis produced too few fully characterized candidates
to support meaningful unsupervised learning. With an insufficient candidate
population, clustering or anomaly-detection methods would have little
underlying structure from which to learn and their results would not provide a
reliable basis for candidate interpretation.

For this reason, unsupervised learning was also deferred until a larger and
more diverse candidate dataset can be constructed.

#### ML Feasibility Conclusion

The project therefore stops short of applying machine learning to the current
candidate dataset. This was a deliberate methodological decision rather than a
technical limitation of the available algorithms.

The preceding phases demonstrate that candidate features can be extracted,
including persistence, run length, spectral bandwidth, frequency drift,
goodness of fit, and other time-frequency characteristics. These measurements
could eventually form a feature space for machine-learning analysis once a
sufficiently large and appropriately validated candidate population is
available.

Future supervised work would require reliable labels and representative
examples of relevant signal classes. Future unsupervised work would require a
substantially larger collection of independently characterized candidates
before clustering or anomaly-detection methods could be meaningfully
evaluated.

Most importantly, statistical anomaly alone is not evidence of extraterrestrial
origin. Machine learning could eventually assist with candidate prioritization
or signal classification, but establishing physical origin would require
additional observational evidence and validation beyond the features extracted
from a single observation.


## Data

The project analyzes radio-telescope time-frequency observations of GJ 144
using two HDF5 data products from the same observation. The smaller `.0002.h5`
product (~263.7 MB) provides greater temporal resolution, while the larger
`.0000.h5` product (~17.33 GB) provides substantially greater frequency
resolution at the cost of temporal resolution.

The raw HDF5 files are not included in this repository because of their size.
Download instructions, source information, filenames, and observation metadata
are provided in [`data/README.md`](data/README.md).


## Analysis Pipeline

The analysis was developed across five stages:

1. **Observation Exploration and Candidate Detection** — Explored the structure
   of the observation and identified statistically unusual frequency features
   for further investigation.

2. **Reusable Candidate Detection Pipeline** — Converted the exploratory
   detection process into a reusable, window-based pipeline for extracting
   structured candidate records from the observation.

3. **Time-Frequency Candidate Characterization** — Examined candidate behavior
   across time and frequency using persistence, run length, spectral bandwidth,
   peak-frequency tracking, linear drift estimation, and sub-bin peak
   interpolation.

4. **High-Resolution Candidate Investigation** — Re-examined the candidate
   using a substantially higher-frequency-resolution data product, revealing
   finer spectral structure including an approximately 500 Hz comb-like
   pattern and enabling additional temporal and drift characterization.

5. **Machine Learning Feasibility** — Evaluated whether the available candidate
   population and ground-truth information were sufficient to support
   supervised or unsupervised machine learning and concluded that additional
   data would be required for a defensible ML analysis.

## Candidate Characterization

The analyzed candidate exceeded the statistical detection threshold and
exhibited measurable structure across both time and frequency. Temporal
analysis showed intermittent persistence, while spectral analysis revealed
changes in peak-frequency location and a relatively stable bandwidth.

Although linear fits produced measurable drift slopes, the corresponding R²
values were low, indicating that the candidate's frequency evolution was not
well described by a coherent linear trajectory.

Higher-frequency-resolution analysis revealed additional structure that was
not resolved in the original data product, including a repeating comb-like
pattern with approximately 500 Hz peak spacing. These results establish the
candidate as a structured and statistically unusual spectral feature, but they
do not establish its physical origin.


## Repository Structure

```text
h5-narrowband-candidate-analysis/
├── README.md
├── requirements.txt
├── data/
│   └── README.md
└── notebooks/
    ├── 01_explore_observation.ipynb
    ├── 02_build_candidates.ipynb
    ├── 03_time_frequency_features.ipynb
    ├── 04_ground_truth_labels.ipynb
    └── 05_ml_feasibility.ipynb
```

## Reproducing the Analysis

### 1. Clone the Repository

```bash
git clone <repository-url>
cd h5-narrowband-candidate-analysis
```

### 2. Create a Python Environment

Creating a dedicated virtual environment is recommended:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

### 3. Install Dependencies

Install the required Python packages:

```bash
pip install -r requirements.txt
```

The primary dependencies are NumPy, pandas, Matplotlib, SciPy, and Blimpy.

### 4. Obtain the Observation Data

The raw HDF5 observation products are not included in this repository because
of their size. Source information, exact filenames, observation metadata, and
download instructions are provided in [`data/README.md`](data/README.md).

After downloading the observation products, create a local `data/raw/`
directory and place the HDF5 files inside it. This directory is excluded from
version control.

The notebooks reference the observations using relative paths of the form:

```text
../data/raw/<observation-file>.h5
```

### 5. Run the Analysis

The notebooks follow the progression of the analysis and should be reviewed or
run in numerical order:

```text
01_explore_observation.ipynb
02_build_candidates.ipynb
03_time_frequency_features.ipynb
04_high_resolution_candidate_analysis.ipynb
05_ml_feasibility.ipynb
```

## Limitations

[Scientific limitations and what cannot be concluded]

## Future Work

[What would logically come next]
