

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
%%capture
%run -t monitor_daily_rotation.ipynb
%store -r MONITOR_TARGET
%store -r stock_df_dict
```


```python
monitor_df = pd.DataFrame(columns=['SYMBOL', 'DATE', 'CLOSE'])

for symbol in MONITOR_TARGET:
    df = stock_df_dict[symbol].iloc[-30:].copy()
    
    df.reset_index(drop=False, inplace=True)
    df['date'] = df['date'].apply(lambda x: x.to_timestamp().to_datetime64())
    df.set_index('date', inplace=True)
    
    today_market = df.iloc[-1]
    now_point = today_market.close
    for col in df.columns:
        if 'ROLLINGMAX' in col:
            buy_point = today_market[col]
            buy_diff = (buy_point - now_point) / now_point * 100
        elif 'ROLLINGMIN' in col:
            sell_point = today_market[col]
            sell_diff = (now_point - sell_point) / now_point * 100
        elif 'MA' in col:
            ma = today_market[col]
            ma_diff = (now_point - ma) / now_point * 100
        elif 'N_chg' in col:
            n_chg = today_market[col] * 100
    
    monitor_df = monitor_df.append({
        'SYMBOL': symbol, 
        'DATE': today_market.name.date(), 
        'CLOSE': now_point, 
        'MA': ma,
        'MADIFF': '%.2f%%' % ma_diff,
        'N_sht': today_market.N_sht, 
        'N_chg': '%.2f%%' % n_chg,
    }, ignore_index=True)
    
    title = '%s, %s, now=%.2f, N_chg=%.2f(%.2f%%), MADIFF=%.2f(%.2f%%)' % \
        (symbol, today_market.name.date(), now_point, today_market.N_sht, n_chg, ma, ma_diff)
    
    df.drop(columns=['N_chg'], inplace=True)
    ax = df.plot(kind='line', title=title, linewidth=0.9, grid=True, figsize=(19, 5))
    ax.yaxis.tick_right()
    
#     display_charts(df, chart_type='stock', kind='line', title=title, figsize=(1000, 600))

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
      <th>CLOSE</th>
      <th>MA</th>
      <th>MADIFF</th>
      <th>N_chg</th>
      <th>N_sht</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>399300</td>
      <td>2019-06-13</td>
      <td>3685.3900</td>
      <td>3637.478824</td>
      <td>1.30%</td>
      <td>1.21%</td>
      <td>3641.180</td>
    </tr>
    <tr>
      <th>1</th>
      <td>000905</td>
      <td>2019-06-13</td>
      <td>4877.6500</td>
      <td>4876.137647</td>
      <td>0.03%</td>
      <td>-0.99%</td>
      <td>4926.400</td>
    </tr>
    <tr>
      <th>2</th>
      <td>399006</td>
      <td>2019-06-13</td>
      <td>1479.8300</td>
      <td>1470.950000</td>
      <td>0.60%</td>
      <td>-0.36%</td>
      <td>1485.240</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BTC</td>
      <td>2019-06-15</td>
      <td>8749.2000</td>
      <td>8097.810000</td>
      <td>7.45%</td>
      <td>4.33%</td>
      <td>8386.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>EOS</td>
      <td>2019-06-15</td>
      <td>6.6760</td>
      <td>6.704833</td>
      <td>-0.43%</td>
      <td>4.28%</td>
      <td>6.402</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ETH</td>
      <td>2019-06-15</td>
      <td>264.5800</td>
      <td>253.171000</td>
      <td>4.31%</td>
      <td>3.39%</td>
      <td>255.900</td>
    </tr>
    <tr>
      <th>6</th>
      <td>XRP</td>
      <td>2019-06-15</td>
      <td>0.4071</td>
      <td>0.406710</td>
      <td>0.10%</td>
      <td>3.32%</td>
      <td>0.394</td>
    </tr>
    <tr>
      <th>7</th>
      <td>LTC</td>
      <td>2019-06-15</td>
      <td>134.4800</td>
      <td>109.654333</td>
      <td>18.46%</td>
      <td>4.98%</td>
      <td>128.100</td>
    </tr>
  </tbody>
</table>
</div>




![png](output_2_1.png)



![png](output_2_2.png)



![png](output_2_3.png)



![png](output_2_4.png)



![png](output_2_5.png)



![png](output_2_6.png)



![png](output_2_7.png)



![png](output_2_8.png)

