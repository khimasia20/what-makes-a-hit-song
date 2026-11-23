# What Makes a Hit Song?

An ECON6113 econometrics project examining whether happier-sounding songs receive more streams after accounting for other audio features and genre.

**Researchers:** Alex Griffa, Preet Khimasia, Shakthi Kannan, and Yusuf Mirasyedi.

## Research question

Are higher-valence tracks associated with more streams after accounting for other audio features and genre?

Valence measures musical positivity on a scale from 0 to 1. The project uses total recorded streams as a measure of popularity and follows a fictional consulting scenario for Aspy Records. The estimates describe associations in the dataset; they do not establish that changing a song's valence would cause additional streams.

## Main findings

The supplied notebook contains the following saved results for 10,000 tracks:

| Specification | Valence coefficient | Estimated change for +0.10 valence | Adjusted R-squared |
| --- | ---: | ---: | ---: |
| Raw-stream OLS | 2,879,733 | +287,973 streams | 0.0027 |
| Simple log-stream OLS | 1.3676 | +14.66% | 0.0146 |
| Log-stream OLS with tempo, acousticness, and duration | 0.9521 | +9.99% | 0.0555 |
| Preferred log-stream model with tempo, acousticness, and genre | 0.5041 | +5.17% | 0.0881 |

Percentage changes refer to **fitted geometric mean streams**, calculated as `100 * (exp(0.10 * beta_valence) - 1)`. They are not percentage changes in the arithmetic mean of streams. Compare model fit only within the same outcome scale.

In the preferred model, +0.10 valence is associated with **5.17% higher fitted geometric mean streams**, with a **95% HC0 confidence interval of 2.53% to 7.88%** and a valence p-value of **0.000101**. Adding genre controls substantially reduces the estimated association.

These figures summarize the notebook's retained outputs. Repository preparation included static validation, not a fresh execution of the analysis.

## Repository contents

| File | Purpose |
| --- | --- |
| [what_makes_a_hit_song.ipynb](what_makes_a_hit_song.ipynb) | Original analysis notebook, including narrative, code, tables, and charts |
| [README.md](README.md) | Project overview, findings, methodology, data information, and run instructions |
| [requirements.txt](requirements.txt) | Python analysis and Jupyter dependencies |
| [.gitignore](.gitignore) | Excludes local environments, notebook checkpoints, credentials, and downloaded data |

The notebook is an unchanged copy of `Final_Project_Econometrics (3).ipynb`, renamed for the repository.

## Data

- **Source:** [What Makes a Song Go Viral? — 10,000 Tracks](https://www.kaggle.com/datasets/vedauv/what-makes-a-song-go-viral-10000-tracks/data) by `vedauv` on Kaggle.
- **Dataset handle:** `vedauv/what-makes-a-song-go-viral-10000-tracks`.
- **File loaded:** `viral_songs.csv`.
- **Saved sample:** 10,000 tracks, 17 columns, and ten genres.

The notebook downloads the dataset with `kagglehub.dataset_load` and the Pandas adapter. The CSV is not bundled in this repository. Consult the source dataset for its provenance, current contents, and reuse terms; the project does not independently verify the underlying measurements.

The principal variables are:

| Variable | Meaning and use |
| --- | --- |
| `streams` | Total recorded streams; the dependent variable, in levels or natural logs |
| `valence` | Musical positivity, from 0 to 1; the main explanatory variable |
| `tempo_bpm` | Tempo in beats per minute; an audio control |
| `acousticness` | Acousticness score, from 0 to 1; an audio control |
| `duration_sec` | Track duration in seconds; examined in sensitivity models |
| `genre` | Nominal genre category; Pop is the reference category |

The dataset also includes track and artist identifiers, release year, additional audio features, and a binary `viral` label. The preferred model does not use `viral` as its outcome. The displayed CSV uses the column name `artist`, although the notebook's introductory schema lists `artist_name`.

## Methodology

The notebook proceeds through descriptive statistics, an unadjusted raw-stream regression, natural-log regressions, additional audio controls, quadratic duration, a duration sensitivity comparison, and a preferred model with genre indicators. Its appendix compares the main specifications using HC0 standard errors.

The preferred specification is:

```python
np.log(streams) ~ valence + tempo_bpm + acousticness + C(genre, Treatment(reference="Pop"))
```

It is fitted by ordinary least squares with heteroskedasticity-robust HC0 standard errors. The final estimation sample drops missing values in `streams`, `valence`, `tempo_bpm`, `acousticness`, `duration_sec`, and `genre`, and keeps positive stream counts. Duration remains part of this common-sample definition even though it is excluded from the preferred regression. With an intercept, nine genre indicators represent the ten categories, and one shared valence slope is estimated across genres.

Duration is omitted from the preferred specification because the same-sample estimate changes only from +5.19% with duration to +5.17% without it, with nearly identical adjusted R-squared. The HC0 p-value for linear duration is 0.331. The earlier quadratic-duration model reports a p-value of 0.1744 for the squared term.

Saved preferred-model diagnostics are:

| Diagnostic | p-value | Interpretation |
| --- | ---: | --- |
| Breusch-Pagan / Koenker | 4.21e-18 | Rejects constant error variance |
| White | 1.34e-13 | Rejects constant error variance |
| Ramsey RESET, HC0 | 0.0784 | Does not reject at the 5% level; this does not prove correct specification |

The preferred model has R-squared 0.0892 and adjusted R-squared 0.0881, leaving much of the variation in streams unexplained.

## Run the notebook

Use Python 3.10 or newer. Python 3.12 is a reasonable starting point; the original notebook records a Python 3 kernel without an exact version.

1. Download or clone this repository and open a terminal in its folder.
2. Create an isolated environment and install the dependencies.

   **Windows PowerShell:**

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\python.exe -m pip install -r requirements.txt
   .\.venv\Scripts\python.exe -m jupyterlab
   ```

   **macOS / Linux:**

   ```bash
   python3 -m venv .venv
   .venv/bin/python -m pip install -r requirements.txt
   .venv/bin/python -m jupyterlab
   ```

3. Open `what_makes_a_hit_song.ipynb` in JupyterLab. Select the environment's Python kernel, restart it, and run all cells from top to bottom.
4. Allow the data-loading cell to download `viral_songs.csv` from Kaggle. The first run requires internet access. If Kaggle requires authentication or consent for the resource, follow the [official KaggleHub setup instructions](https://github.com/Kaggle/kagglehub#authenticate).

The requirements include `kagglehub[pandas-datasets]` for the [Pandas dataset adapter](https://github.com/Kaggle/kagglehub#load-dataset). `statsmodels>=0.14.6` covers the warning class imported by the diagnostics cell. The dependency file is a setup list, not a lockfile or a reconstruction of the original runtime.

For an offline rerun, first obtain the CSV from the source and place it beside the notebook. Replace the dataset-loading cell locally with:

```python
df_songs = pd.read_csv("viral_songs.csv")
display(df_songs.head())
```

## Reproducibility and limitations

- The dataset loader requests the latest dataset version. A future rerun can differ if the source changes; record the version and package versions for a reproducible research run.
- The original notebook retains outputs but has null execution counts. Its saved tables should be distinguished from a newly verified run.
- Later cells reuse models and variables created earlier. Run the whole notebook in order, particularly the duration comparison before the diagnostics and appendix.
- Earlier exploratory log regressions assume positive stream counts; the final estimation sample filters them explicitly.
- Artist reach, marketing, playlist placement, and time since release are not controlled for in the preferred model. These factors could affect both song characteristics and streams.
- Additional audio controls and genre-specific valence slopes could change the estimates. Statistical significance alone does not imply strong predictive performance or a causal effect.
- Some internal appendix links use Colab-specific anchors and may not navigate correctly in GitHub or JupyterLab.

## Credits

Prepared for ECON6113 by Alex Griffa, Preet Khimasia, Shakthi Kannan, and Yusuf Mirasyedi. Dataset credit belongs to its Kaggle publisher. This repository does not assign a new license to the dataset or the coauthored project.
