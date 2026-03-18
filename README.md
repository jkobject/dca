## Deep count autoencoder for denoising scRNA-seq data

A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect by taking the count structure, overdispersed nature and sparsity of the data into account using a deep autoencoder with zero-inflated negative binomial (ZINB) loss function.

See our [manuscript](https://www.nature.com/articles/s41467-018-07931-2) and [tutorial](https://nbviewer.ipython.org/github/theislab/dca/blob/master/tutorial.ipynb) for more details.

### Installation

#### pip

For a traditional Python installation of the count autoencoder and the required packages, use

```
$ pip install dca
```

#### conda

Another approach for installing count autoencoder and the required packages is to use [Conda](https://conda.io/docs/) (most easily obtained via the [Miniconda Python distribution](https://conda.io/miniconda.html)). Afterwards run the following commands.

```
$ conda install -c bioconda dca
```

### Usage

You can run the autoencoder from the command line:

`dca matrix.csv results`

where `matrix.csv` is a CSV/TSV-formatted raw count matrix with genes in rows and cells in columns. Cell and gene labels are mandatory. 

### Results

Output folder contains the main output file (representing the mean parameter of ZINB distribution) as well as some additional matrices in TSV format:

- `mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution. This file has the same dimensions as the input file (except that the zero-expression genes or cells are excluded). It is formatted as a `gene x cell` matrix. Additionally, `mean_norm.tsv` file contains the library size-normalized expressions of each cell and gene. See `normalize_total` function from [Scanpy](https://scanpy.readthedocs.io/en/stable/api/scanpy.pp.normalize_total.html) for the details about the default library size normalization method used in DCA.

- `pi.tsv` and `dispersion.tsv` files represent dropout probabilities and dispersion for each cell and gene. Matrix dimensions are same as `mean.tsv` and the input file.

- `reduced.tsv` file contains the hidden representation of each cell (in a 32-dimensional space by default), which denotes the activations of bottleneck neurons.

Use `-h` option to see all available parameters and defaults.

### Hyperparameter optimization

You can run the autoencoder with `--hyper` option to perform hyperparameter search.

---

### Fork notes (jkobject/dca)

This is a fork of [theislab/dca](https://github.com/theislab/dca) maintained for use as a submodule in [scPRINT-2](https://github.com/cantinilab/scPRINT-2).

#### What was changed and why

DCA was written for Keras 1.x / early Keras 2.x and uses several APIs that have been removed or relocated in modern versions of Keras (2.12+):

| File | Old import | New import | Reason |
|------|-----------|------------|--------|
| `dca/network.py` | `from keras.objectives import mean_squared_error` | `from keras.losses import mean_squared_error` | `keras.objectives` module was removed; losses moved to `keras.losses` |
| `dca/layers.py` | `from keras.engine.topology import Layer` | `from keras.layers import Layer` | `keras.engine.topology` was deprecated and removed; `Layer` is now directly in `keras.layers` |
| `dca/layers.py` | `from keras.engine.base_layer import InputSpec` | `from keras.layers import InputSpec` | Same — `keras.engine.base_layer` no longer exists |

These are minimal, non-breaking patches — no logic was changed, only import paths updated to match Keras 2.12 conventions.

#### Usage in scPRINT-2

```python
import sys
sys.path.insert(0, "tools/dca")
from dca.api import dca
result = dca(adata, copy=True)
```

Requires a separate Python environment with `tensorflow==2.12.0` and `keras==2.12.0` (incompatible with the main scPRINT-2 environment due to TF version conflicts).
