# wolfquant
重新构建集成交易、回测、建模的AI投资框架。
期货接口基于pyctp，使用语言python3.6，环境linux64/ubuntu

# 使用
## 交易接口调用
1.安装包
```shell
$ python setup.py
```
2.复制配置文件，更新配置信息
```shell
$ cp etc/config-default.json config.json
```
3.使用案例
```python
# 通过以下方式使用期货版API
>>>from wolfquant.api.future import ApiStruct, MdApi, TraderApi
```
4.运行测试案例
```shell
$ cd tests
$ python test_api.py
```
## 因子构建并采用机器学习模型训练
```python
import pandas as pd
import numpy as np
import tushare as ts
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis, QuadraticDiscriminantAnalysis
from sklearn.svm import LinearSVC, SVC
from sklearn.metrics import confusion_matrix
from wolfquant.utils.factor_utils import Factor_pipeline
from wolfquant.factors import trade_factors as tf

# 获取数据
datasets = ts.get_k_data('000300', start='2005-01-01', index=True).set_index('date')
datasets.index = pd.to_datetime(datasets.index, format='%Y-%m-%d')

# 构建特征
datasets = Factor_pipeline(datasets).add(tf.SMA, 50)\
                                    .add(tf.EWMA, 200)\
                                    .add(tf.BBANDS, 50)\
                                    .add(tf.CCI, 20)\
                                    .add(tf.ForceIndex, 1)\
                                    .add(tf.EVM, 14)\
                                    .add(tf.ROC, 5)\
                                    .add(tf.LAGRETURN, 0)\
                                    .data.dropna()

# 构建标签
datasets['direction'] = np.sign(datasets['LAGRETURN_0']).shift(-1)
datasets = datasets.dropna()

# 构建训练集和测试集
X = datasets[datasets.columns[6:-2]]
y = datasets['direction']
start_test = '2012-01-01'
X_train = X.loc[:start_test]
X_test = X.loc[start_test:]
y_train = y.loc[:start_test]
y_test = y.loc[start_test:]

# 构建模型
print('Hit rates/Confusion Matrices:\n')
models = [('LR', LogisticRegression()),
          ('LDA', LinearDiscriminantAnalysis()),
          ('QDA', QuadraticDiscriminantAnalysis()),
          ('LSVC', LinearSVC()),
          ('RSVM', SVC()),
          ('RF', RandomForestClassifier(n_estimators=1000))]

# 遍历模型
for m in models:
    # 训练模型
    m[1].fit(X_train, y_train)
    # 预测模型
    pred = m[1].predict(X_test)
    # 输出hit-rate和交叉验证结果
    print("%s:\n%0.3f" % (m[0], m[1].score(X_test, y_test)))
    print("%s\n" % confusion_matrix(pred, y_test))
```

# 路线图
### 0.0.0
* 实现了期货python版的交易接接口
* 整理交易接口的使用文档
### 0.0.1
* 添加交易接口的测试案例
### 0.0.2
* 期货交易接口二次开发。
* 添加feature处理模块。
### 0.0.3
* 添加回测模块

# 附言
该项目会长期做，有志同道合的小伙伴，欢迎一起入坑，我的微信号wolfquant。
