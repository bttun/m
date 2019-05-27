

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



<script src="//code.highcharts.com/stock/highstock.js"></script>
<script src="//code.highcharts.com/highcharts-more.js"></script>
<script src="//code.highcharts.com/modules/exporting.js"></script>




<style>.container { width:70% !important; }</style>



```python
%store -r MONITOR_TARGET
%store -r stock_df_dict
```


```python
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
    title = '%s, %s, open=%d, buy=%d(+%.1f%%), sell=%d(-%.1f%%)' % \
        (symbol, today_market.name.date(), now_point, buy_point, buy_diff, sell_point, sell_diff)
    
#     display_charts(df, chart_type='stock', kind='line', title=title, figsize=(1000, 600))
    ax = df.plot(kind='line', title=title, linewidth=0.9, grid=True, figsize=(19, 7))
    ax.yaxis.tick_right()
```


![png](output_2_0.png)



![png](output_2_1.png)



![png](output_2_2.png)



![png](output_2_3.png)



![png](output_2_4.png)

