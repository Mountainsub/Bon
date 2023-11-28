# Bond_and_Reit5 

※2023/08/03 追記中   

https://github.com/Mountainsub/Bond_and_Reit4  
This directory  is succeed.  
I suppose US REIT ETF trading, as trade liquidity.



### Back Test5 
X-Train data  : Receipts, Outlays-Receipts, Government Account Series, Local and State Account Series  
Y-Train Data : {0, 1} Label, US Bond(yield) increment sign  
symbol : IYR, USBond Future  
Train Term : 2001-01 ~ 2023-06, interval=2month  
Test Term : 2011-01 ~ 2023-06, interval=2month  
startdate in a horizon :  12th day  
enddate in a horizon :  11th day  

Machine Learning ; SVM Algorithm(SVC with radical base function, as rbf,  Regression) 
...  

シャープレシオ： 1.74(odd month start)  0.98(even month start)  

ドローダウン  
奇数月スタート(1, 3, 5, 7, 9, 11)
![graph_2month_oddstart](https://github.com/Mountainsub/Bond_and_Reit5/assets/70077254/4fe777d4-2d91-4d7a-bbd2-c675d845725c)  
偶数月スタート(2, 4, 6, 8, 10, 12)  
![graph_2month_evenstart](https://github.com/Mountainsub/Bond_and_Reit5/assets/70077254/4ec39d68-fda0-48c6-91f9-156dc25fc228)  

USBond Future  
  
 


### 変更点(日本語)　
インターバルを2か月にすると、オプションが無かった方がパフォーマンスが良かったため、廃止

```python
np.where(US_Bond[1:] - US_Bond[0:-1] + option , 1, 0)  
```

folding, folding_price_data  
Explanation : in case of multi-month interval, it folds traindata-dataframe or price-dataframe.  
```python
def folding_price_data(df, index2):
       


    DATA = pd.DataFrame({})


    for i in range(0,len(df), index2):

        try:
            temp = df.iloc[i:index2+i,:]
        except:
            temp = df.iloc[i:,:]
        M, m = max(temp["High"]), min(temp["Low"])
        open_p, close_p = temp['Open'].values[0], temp['Close'].values[-1]
        tp = pd.DataFrame({'Open':[open_p], 'High':[M], 'Low':[m], 'Close':[close_p]}, index=[temp.index.to_list()[0]])
        DATA = pd.concat([DATA, tp], axis=0)
        

    #DATA.index = df.query('@df.index.month % @index2== 0').index

    return DATA
```

Evaluation : 分析結果のデータフレームと利益分布のヒストグラムを表出  
```python
def Evaluation(df):
    Sel = pd.Series()
    Sel['初期資金'] = df['Asset'][0]
    Sel['必要最低資金'] = df['Asset'].max()
    Sel["全トレード数"] = df.shape[0]
    #print((10**2) * df.query('interest >= 0').count() / df.shape[0])
    Sel["勝率"] = "{:.2f}%".format((10**2) * df.query('interest >= 0').shape[0] / df.shape[0])
    Sel["負率"] = "{:.2f}%".format((10**2)* df.query('interest < 0').shape[0] / df.shape[0])
    Sel["全トレード平均利益"] = df['interest'].mean()
    Sel["勝ちトレード平均利益"] =  df.query('interest >= 0')['interest'].mean()
    Sel["負けトレード平均利益"] =  df.query('interest < 0')['interest'].mean()
    
    Sel["プロフィットファクター"] = df.query('interest >= 0')['interest'].sum() / -df.query('interest < 0')['interest'].sum()
    
    display(pd.DataFrame(Sel))
    fig, ax = plt.subplots()
    ax.hist(df['interest'], bins=20, label='interest')

```
Others  

計算用プログラムを一部編集。Tail_or_notを削除、無駄な部分削除、calcの元位置の第5因数(lever)削除(毎回Trueなので。)等。　　
