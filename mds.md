

```python
import pandas as pd
pd.core.common.is_list_like = pd.api.types.is_list_like

from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = 'all'

from pandas_highcharts.core import serialize
from pandas_highcharts.display import display_charts

import matplotlib
import matplotlib.pyplot as plt

from IPython.core.display import display, HTML
display(HTML("<style>.container { width:70% !important; }</style>"))
```



<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
<script src="https://code.highcharts.com/stock/highstock.js"></script>
<script src="https://code.highcharts.com/stock/modules/exporting.js"></script>
<script src="https://code.highcharts.com/stock/modules/export-data.js"></script>




<style>.container { width:70% !important; }</style>



```python
%store -r MONITOR_TARGET
%store -r stock_df_dict
```


```python
monitor_df = pd.DataFrame(columns=['SYMBOL', 'DATE', 'NOW', 'BUY', 'BUYDIFF', 'SELL', 'SELLDIFF'])

for symbol in MONITOR_TARGET:
    df = stock_df_dict[symbol].iloc[-100:].copy()    
    df.reset_index(drop=False, inplace=True)
    # df = df.astype(dtype={'date': 'datetime64[ns]'})
    df['date'] = df['date'].apply(lambda x: x.to_timestamp().to_datetime64())
    df.set_index('date', inplace=True)

    today_market = df.iloc[-1]
    now_point = today_market.open
    for col in df.columns:
        if 'ROLLINGMAX' in col:
            buy_point = today_market[col]
            buy_diff = (buy_point - now_point) / now_point * 100
        elif 'ROLLINGMIN' in col:
            sell_point = today_market[col]
            sell_diff = (now_point - sell_point) / now_point * 100
    
    monitor_df = monitor_df.append({
        'SYMBOL': symbol, 
        'DATE': today_market.name.date(), 
        'NOW': now_point, 
        'BUY': buy_point, 
        'BUYDIFF': '+%.2f%%' % buy_diff, 
        'SELL': sell_point, 
        'SELLDIFF': '-%.2f%%' % sell_diff,
    }, ignore_index=True)
    
    title = '%s, %s, open=%d, buy=%d(+%.1f%%), sell=%d(-%.1f%%)' % \
        (symbol, today_market.name.date(), now_point, buy_point, buy_diff, sell_point, sell_diff)
#     display_charts(df, chart_type='stock', kind='line', title=title, figsize=(1000, 600))
    ax = df.plot(kind='line', title=title, linewidth=0.9, grid=True, figsize=(19, 7))
    ax.yaxis.tick_right()
    
monitor_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SYMBOL</th>
      <th>DATE</th>
      <th>NOW</th>
      <th>BUY</th>
      <th>BUYDIFF</th>
      <th>SELL</th>
      <th>SELLDIFF</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>399300</td>
      <td>2019-05-28</td>
      <td>3633.02</td>
      <td>4125.97</td>
      <td>+13.57%</td>
      <td>3584.90</td>
      <td>-1.32%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>000905</td>
      <td>2019-05-28</td>
      <td>4965.59</td>
      <td>5821.02</td>
      <td>+17.23%</td>
      <td>4848.58</td>
      <td>-2.36%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>399006</td>
      <td>2019-05-28</td>
      <td>1497.12</td>
      <td>1787.96</td>
      <td>+19.43%</td>
      <td>1447.98</td>
      <td>-3.28%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BITCOIN</td>
      <td>2019-05-27</td>
      <td>8674.07</td>
      <td>8674.07</td>
      <td>+0.00%</td>
      <td>7267.96</td>
      <td>-16.21%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>EOS</td>
      <td>2019-05-27</td>
      <td>6.90</td>
      <td>6.90</td>
      <td>+0.00%</td>
      <td>5.89</td>
      <td>-14.64%</td>
    </tr>
  </tbody>
</table>
</div>




![png](output_2_1.png)



![png](output_2_2.png)



![png](output_2_3.png)



![png](output_2_4.png)



![png](output_2_5.png)

