# pvcurve

[![PyPI version](https://badge.fury.io/py/pvcurve.svg)](https://pypi.org/project/pvcurve/)

A lightweight python package for analyzing pressure-volume curves. `pvcurve` provides a simple, fast way to calculate common PV curve metrics (e.g., turgor loss point, capacitance, osmotic potential) with automated turgor loss detection, built-in plotting, and optional outlier detection.

The underlying calculations are based on the Williams P-V Curve analyzer (see References). This package was originally developed for [a project I've been working on](https://github.com/jean-allen/spectral_pv_curves) and is shared in the hope that it can be useful to other folks too. ☺

## Installation

Install from PyPI:

```bash
pip install pvcurve
```

Or install directly from GitHub:
```bash
pip install git+https://github.com/jean-allen/pvcurve
```

## Quick start

```python
import pvcurve as pvc
my_pv_curve = pvc.read('my_file.xlsx')
print(my_pv_curve)
my_pv_curve.plot()
```

![Example output of my_pv_curve.plot()](https://github.com/jean-allen/pvcurve/blob/main/tests/output_img.png?raw=true)

Example data is available under `tests/eugl_1.xlsx`, along with a notebook (`reading_in_file.ipynb`) that walks through the use cases seen below in a more interactive format

A data collection template is also available at `templates/data_collection_template.xlsx`. If `pvcurve` has trouble reading your data sheet automatically, the fastest solution is probably to copy your data into that spreadsheet. (Also consider [opening an issue](https://github.com/jean-allen/pvcurve/issues) to let me know what issue you're having!)

## Reading data

`pvcurve` is designed to parse files automatically and should be able to interpret both CSV and Excel files, so long as the column keys are in the first row of the spreadsheet:

```python
import pvcurve as pvc
my_pv_curve = pvc.read('my_data.csv')
```

Column mapping (matching columns to Ψ/mass/dry mass data) is inferred automatically by default. If this process fails or picks up the wrong column, you can specify what column contains what explicitly:

```python
my_pv_curve = pvc.read(
    'my_data.csv',
    psi_column="Water Potential",
    mass_column="Wet Mass",
    dry_mass_column="Dry Mass"
)
```

Similarly, units will be inferred from the column names when possible but can also be specified explicitly:

```python
my_pv_curve = pvc.read(
    'my_data.csv',
    psi_column="Water Potential",
    mass_column="Wet Mass",
    dry_mass_column="Dry Mass",
    psi_units="bar",
    mass_units="mg"
)
```

The breakpoint for what data is considered to be "pre-TLP" and "post-TLP" is detected automatically based on the weighted R² of the before/after TLP regressions (same as the Williams P-V Analyzer, just a little more automated). You can visualize the breakpoint selection and how it affects your results by calling the `get_breakpoint` function:

```python
breakpoint = my_pv_curve.get_breakpoint(plot=True)
```
![Example output of my_pv_curve.get_breakpoint(plot=True)](https://github.com/jean-allen/pvcurve/blob/main/tests/breakpoint.png?raw=true)

## Outlier detection and removal

For datasets with significant transcription errors or noisy measurements, `pvcurve` includes two automated outlier detection algorithms for you to experiment with. The default method identifies outliers based on a confidence interval around the before/after TLP regressions (by default using a 95% confidence interval, as below):

```python
pv_clean = pv.remove_outliers(method="regression")
```

A secondary, somewhat experimental method detects points that deviate strongly from the local median, calculated using a moving window, for both Ψ and wet mass. This approach is less prone to false positives, but has been less extensively tested, so use only with appropriate caution:

```python
pv_clean = pv.remove_outliers(method="local_mad")
```

## Calculating values and visualizing output

All derived metrics (e.g., TLP) are calculated when the `PVCurve` object is created and  stored as attributes:

```python
print('Turgor Loss Point (MPa):', my_pv_curve.tlp)
print('Turgor Loss Point Confidence Interval (MPa):', my_pv_curve.tlp_conf_int)
print('Saturated Water Content:', my_pv_curve.swc)
print('Bulk elastic modulus (MPa):', my_pv_curve.bulk_elastic_total)
```

Plotting is available using the `plot` function:

```python
my_pv_curve.plot()
```

## Saving data

Results and all calculated values can be exported to CSV or Excel

```python
my_pv_curve.save_csv("results.csv")
my_pv_curve.save_excel("results.xlsx")
```

## References

The science underlying this package is drawn from the following works:

Bartlett, M.K., Scoffoni, C., Sack, L. (2012). The determinants of leaf turgor loss point and prediction of drought tolerance of species and biomes: a global meta-analysis. Ecology Letters 15: 393-405. https://doi.org/10.1111/j.1461-0248.2012.01751.x

Hinckley, T.M., Duhme, F., Hinckley, A.R., Richter, H. (1980). Water relations of drought hardy shrubs: osmotic potential and stomatal reactivity. Plant, Cell & Environment 3: 131-140. https://doi.org/10.1111/1365-3040.ep11580919

Koide, R.T., Robichaux, R.H., Morse, S.R., Smith, C.M. (1989). Plant water status, hydraulic resistance and capacitance. In: Pearcy, R.W., Ehleringer, J.R., Mooney, H.A., Rundel, P.W. (eds) Plant Physiological Ecology. Springer, Dordrecht. https://doi.org/10.1007/978-94-009-2221-1_9

Sack, L., Cowan, P.D., Jaikumar, N., Holbrook, N.M. (2003). The ‘hydrology’ of leaves: co-ordination of structure and function in temperate woody species. Plant, Cell & Environment 26: 1343-1356. https://doi.org/10.1046/j.0016-8025.2003.01058.x

Scholz, F.G., Phillips, N.G., Bucci, S.J., Meinzer, F.C., Goldstein, G. (2011). Hydraulic capacitance: biophysics and functional significance of internal water sources in relation to tree size. In: Meinzer, F.C., Lachenbruch, B., Dawson, T.E. (eds) Size-and age-related changes in tree structure and function. Springer, Dortrecht, The Netherlands, pp 341–361.

Tyree, M.T., Hammel, H.T. (1972). The Measurement of the Turgor Pressure and the Water Relations of Plants by the Pressure-bomb Technique. Journal of Experimental Botany 23: 267–282. https://doi.org/10.1093/jxb/23.1.267