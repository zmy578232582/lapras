

# LAPRAS

[![PyPi version][pypi-image]][pypi-url]
[![Python version][python-image]][docs-url]




Lapras is developed to facilitate the dichotomy model development work.


## Install


via pip

```bash
pip install lapras
```

via source code

```bash
python setup.py install
```

## Usage

```python
import pandas as pd
import lapras

# read data file as pandas dataframe
df = pd.read_csv('data/model_data.csv',encoding="utf-8")
to_drop = ['id']
target = 'bad'

# feature selection
lapras.detect(df.drop(to_drop,axis=1))
lapras.quality(df.drop(to_drop,axis=1),target = target)
train_selected, dropped = lapras.select(df.drop(to_drop,axis=1),target = target, empty = 0.9, \
                                                iv = 0.02, corr = 0.7, return_drop=True, exclude=[])

# bins   method = ['dt', 'kmeans', 'step', 'quantile']                                    
c = lapras.Combiner()
c.fit(train_selected, y = target, method = 'dt', min_samples = 0.05,n_bins=8)
c.export()
# c.load({}) # export the default bins and change it as you wish, finally load it back and take effects.


# bins visualization
cols = train_selected.columns
for col in cols:
    if col != target:
        lapras.bin_plot(c.transform(train_selected[[col,target]], labels=True), col=col, target=target)
        
# transfer to WOE
transfer = lapras.WOETransformer()
train_woe = transfer.fit_transform(c.transform(train_selected), train_selected[target], exclude=[target])

# stepwise method to choose features
final_data = lapras.stepwise(train_woe,target = target, estimator='ols', direction = 'both', criterion = 'aic', exclude = [])

# ScoreCard fit, predict and export
card = lapras.ScoreCard(
    combiner = c,
    transfer = transfer,
)
col = list(final_data.drop([target],axis=1).columns)
card.fit(final_data[col], final_data[target])
final_data['score'] = card.predict(final_data[col])
final_data['prob'] = card.predict_prob(final_data[col])
card.export()


#  performance
lapras.perform(final_data['prob'],final_data[target])
lapras.score_plot(final_data,score='score', target=target)
 

```

## Documents

A simple API.

[pypi-image]: https://img.shields.io/badge/pypi-V0.0.14-%3Cgreen%3E
[pypi-url]: https://github.com/yhangang/lapras
[python-image]: https://img.shields.io/pypi/pyversions/toad.svg?style=flat-square
[docs-url]: https://github.com/yhangang/lapras


