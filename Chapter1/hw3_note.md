# HW3

初始為random forest -> 效果很爛



## Feature Selection

#### GradientBoostingClassifier

不知為何這個方法放到feature select 都會噴錯，pred不是全部都1就全部都0...

#### ExtraTreesClassifier

這個方法比較穩，但配上xgboost 還是只有0.25多，比不上decision tree

#### DecisionTreeClassifier

目前試過最好的，不論是配 GradientBoostingClassifier[0.30639]or XGBOOST[0.32274]，都有不錯的成績。

想用pipeline解決 feature selection問題：

```python
from sklearn.pipeline import Pipeline

model = Pipeline([
  ('feature_selection', SelectFromModel(fea_sel)),
  ('classification', AdaBoostClassifier())])
 model.fit(X_resampled, y_resampled)
  
 pred = model.predict(X_newtest)
  
>>ValueError: X has a different shape than during fitting
  
```

無法fit  X_newtest （此為經過feature selection後的) 不知道原因，可能是feature數量等不同，沒辦法套到model



## Over / Under Sampling

目前只測試oversampling

但在adaboost 下 ，用兩種比較，over結果較好，直接變成0.30086。



## Model Select

#### xgboost 據說是kaggle神器

一開始無法使用，要跑很久跑不出來，不知道是不是電腦的問題

decision tree + xgboost [0.32274]

但和GradientBoostingClassifier() 出來結果都變成0 

#### Adaboost

為一開始效果最好的model，只要加了oversampling，瞬間輕鬆變成0.30086

和GradientBoostingClassifier() 出來結果都變成1

#### Decision tree

加了over sampler 可以過baseline 70 效果尚可，但看起來用在特徵選擇比較好。



### Reference

<http://www.python88.com/topic/24855>

