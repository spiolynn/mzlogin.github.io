---
layout:     post
title:      002-google-wide&depth-结构化分类回归模型
subtitle:    "\"002-google-wide&depth-结构化分类回归模型\""
date:       2018-11-16
author:     PZ
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - ML
    - Wide&Depth
---

## 002-google-wide&depth-结构化分类回归模型

[TOC]

> 完成一个收入预测模型，尝试使用传统`wide` LR 模型 `depth` DNN 模型 和 `wide and depth` 模型

> 通过对人的 年龄、学历、教育程度、职业... 预测收入

- 数据含义: 

```
age: 年龄                   | w 
workclass：                 | w | e
fnlwgt: 代表人数
education：学历             | w | e
education_num： 学习年限
marital_status：婚姻状况
occupation：职业            | w | e
relationship：关系          | w | e
race：
gender : 性别               | w 
capital_gain：资本收益
capital_loss：资本loss
hours_per_week：周工时      | w
native_country：国籍        | w | e
income_bracket：收入等级

education-occupation w-组合特征
native_country-occupation w-组合特征
age-hours_per_week e-组合特征
```

### 1 wide 模型

#### 1 模型结构

![image.png](https://upload-images.jianshu.io/upload_images/14744153-d0809b5b503abda2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2 代码实现

- TensorFlow 1.10.0
- Keras 2.2.2
- python 3.6

> http://mlr.cs.umass.edu/ml/machine-learning-databases/adult/adult.data<br> http://mlr.cs.umass.edu/ml/machine-learning-databases/adult/adult.test


```python
'''
使用LR模型
'''

##################
DEBUG = True
import pandas as pd
from sklearn.preprocessing import OneHotEncoder, MinMaxScaler, StandardScaler
from keras.layers import Dense
from keras.optimizers import Adam
from keras.layers import Input, concatenate, Embedding, Reshape
from keras.models import Model
from keras.utils import plot_model

# step1：load data
COLUMNS = ["age", "workclass", "fnlwgt", "education", "education_num",
           "marital_status", "occupation", "relationship", "race", "gender",
           "capital_gain", "capital_loss", "hours_per_week", "native_country",
           "income_bracket"]

df_train = pd.read_csv("adult.data.txt",names=COLUMNS, skipinitialspace=True)
df_test = pd.read_csv("adult.test.txt",names=COLUMNS, skipinitialspace=True, skiprows=1)
df_train['income_label'] = (
    df_train["income_bracket"].apply(lambda x: ">50K" in x)).astype(int)
df_test['income_label'] = (
    df_test["income_bracket"].apply(lambda x: ">50K" in x)).astype(int)

if DEBUG:
    print('------------------------')
    print(df_train.head(n=10))
    print(df_test.head(n=10))

# step2:
# wide model used cols
wide_cols = ['age','hours_per_week','education', 'relationship', 'workclass',
             'occupation','native_country','gender']
x_cols = (['education', 'occupation'], ['native_country', 'occupation'])
target = 'income_label'
method = "logistic"
model_type ="wide"



def cross_columns(x_cols):
    """ 构建组合特征 """
    crossed_columns = dict()
    colnames = ['_'.join(x_c) for x_c in x_cols]
    for cname, x_c in zip(colnames, x_cols):
        crossed_columns[cname] = x_c
    return crossed_columns

'''
wide 模型
'''
def wide(df_train, df_test, wide_cols, x_cols, target, model_type, method):
    pass
    df_train['IS_TRAIN'] = 1
    df_test['IS_TRAIN'] = 0
    df_wide = pd.concat([df_train, df_test],axis=0)

    # 组合特征
    crossed_columns_d = cross_columns(x_cols)
    wide_cols += crossed_columns_d.keys()
    print('add 组合特征的 wide_cols')
    print(wide_cols)

    # 非数字类特征，需要onehot
    categorical_columns = list(df_wide.select_dtypes(include=['object']).columns)

    for k, v in crossed_columns_d.items():
        df_wide[k] = df_wide[v].apply(lambda x: '-'.join(x), axis=1)
    # print(df_wide.head(n=10))

    # 抽取训练所需数据
    df_wide = df_wide[wide_cols + [target] + ['IS_TRAIN']]

    # 需要进行onehot处理的特征（非数字类特征+组合特征）
    dummy_cols = [
        c for c in wide_cols if c in categorical_columns + list(crossed_columns_d.keys())]
    print('需要onehot特征')
    print(dummy_cols)

    #onehot
    df_wide = pd.get_dummies(df_wide, columns=[x for x in dummy_cols])
    print(type(df_wide))
    print(df_wide.head(n=2))

    # 删除列（axis=1）
    train = df_wide[df_wide.IS_TRAIN == 1].drop('IS_TRAIN', axis=1)
    test = df_wide[df_wide.IS_TRAIN == 0].drop('IS_TRAIN', axis=1)

    # make sure all columns are in the same order and life is easier
    cols = [target] + [c for c in train.columns if c != target]
    train = train[cols]
    test = test[cols]

    #除了第一列
    X_train = train.values[:, 1:]
    #第一列
    y_train = train.values[:, 0].reshape(-1, 1)
    X_test = test.values[:, 1:]
    y_test = test.values[:, 0].reshape(-1, 1)

    # if method == 'multiclass':
    #     y_train = onehot(y_train)
    #     y_test = onehot(y_test)

    # Scaling 归一化（0,1）
    scaler = MinMaxScaler()
    X_train = scaler.fit_transform(X_train)
    X_test  = scaler.fit_transform(X_test)

    activation, loss, metrics = ('sigmoid', 'binary_crossentropy', 'accuracy')

    if metrics:
        metrics = [metrics]
    # simply connecting the features to an output layer
    wide_inp = Input(shape=(X_train.shape[1],), dtype='float32', name='wide_inp')
    w = Dense(y_train.shape[1], activation=activation)(wide_inp)
    wide = Model(wide_inp, w)
    wide.compile(Adam(0.01), loss=loss, metrics=metrics)

    print(wide.summary())
    plot_model(wide, to_file='model_wide.png', show_shapes=True)

    wide.fit(X_train, y_train, nb_epoch=10, batch_size=64)
    results = wide.evaluate(X_test, y_test)
    print("\n", results)

wide(df_train, df_test, wide_cols, x_cols, target, model_type, method)

```

### 2 Depth模型

#### 1 模型结构

![model_deep.png](https://upload-images.jianshu.io/upload_images/14744153-2500808c91c1bc24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2 代码实现

```python
"""
DNN模型
"""

##################
DEBUG = True
import pandas as pd
from sklearn.preprocessing import OneHotEncoder, MinMaxScaler, StandardScaler
from keras.layers import Dense
from keras.optimizers import Adam
from keras.layers import Input, concatenate, Embedding, Reshape
from keras.utils import plot_model
import numpy as np
from keras.layers import merge, Flatten, merge, Lambda, Dropout
from keras.layers.normalization import BatchNormalization
from keras.models import Model
from keras.regularizers import l2, l1_l2

# step1：load data
COLUMNS = ["age", "workclass", "fnlwgt", "education", "education_num",
           "marital_status", "occupation", "relationship", "race", "gender",
           "capital_gain", "capital_loss", "hours_per_week", "native_country",
           "income_bracket"]

df_train = pd.read_csv("adult.data.txt",names=COLUMNS, skipinitialspace=True)
df_test = pd.read_csv("adult.test.txt",names=COLUMNS, skipinitialspace=True, skiprows=1)
df_train['income_label'] = (
    df_train["income_bracket"].apply(lambda x: ">50K" in x)).astype(int)
df_test['income_label'] = (
    df_test["income_bracket"].apply(lambda x: ">50K" in x)).astype(int)

# step2:
# columns for deep model
embedding_cols = ['education', 'relationship', 'workclass', 'occupation',
                  'native_country']
cont_cols = ["age", "hours_per_week"]
target = 'income_label'
method = "logistic"
model_type ="depth"

def cross_columns(x_cols):
    """ 构建组合特征 """
    crossed_columns = dict()
    colnames = ['_'.join(x_c) for x_c in x_cols]
    for cname, x_c in zip(colnames, x_cols):
        crossed_columns[cname] = x_c
    return crossed_columns

def val2idx(df, cols):
    """helper to index categorical columns before embeddings.
    """
    val_types = dict()
    for c in cols:
        val_types[c] = df[c].unique()

    val_to_idx = dict()
    for k, v in val_types.items():
        val_to_idx[k] = {o: i for i, o in enumerate(val_types[k])}

    for k, v in val_to_idx.items():
        df[k] = df[k].apply(lambda x: v[x])

    unique_vals = dict()
    for c in cols:
        unique_vals[c] = df[c].nunique()

    return df, unique_vals

def onehot(x):
    return np.array(OneHotEncoder().fit_transform(x).todense())


def embedding_input(name, n_in, n_out, reg):
    inp = Input(shape=(1,), dtype='int64', name=name)
    return inp, Embedding(n_in, n_out, input_length=1, embeddings_regularizer=l2(reg))(inp)


def continous_input(name):
    inp = Input(shape=(1,), dtype='float32', name=name)
    return inp, Reshape((1, 1))(inp)

def deep(df_train, df_test, embedding_cols, cont_cols, target, model_type, method):

    df_train['IS_TRAIN'] = 1
    df_test['IS_TRAIN'] = 0
    df_deep = pd.concat([df_train, df_test])

    deep_cols = embedding_cols + cont_cols
    print(deep_cols)

    df_deep, unique_vals = val2idx(df_deep, embedding_cols)

    train = df_deep[df_deep.IS_TRAIN == 1].drop('IS_TRAIN', axis=1)
    test = df_deep[df_deep.IS_TRAIN == 0].drop('IS_TRAIN', axis=1)

    embeddings_tensors = []
    n_factors = 8
    reg = 1e-3
    for ec in embedding_cols:
        layer_name = ec + '_inp'
        t_inp, t_build = embedding_input(
            layer_name, unique_vals[ec], n_factors, reg)
        embeddings_tensors.append((t_inp, t_build))
        del(t_inp, t_build)

    continuous_tensors = []
    for cc in cont_cols:
        layer_name = cc + '_in'
        t_inp, t_build = continous_input(layer_name)
        continuous_tensors.append((t_inp, t_build))
        del(t_inp, t_build)

    X_train = [train[c] for c in deep_cols]
    y_train = np.array(train[target].values).reshape(-1, 1)
    X_test = [test[c] for c in deep_cols]
    y_test = np.array(test[target].values).reshape(-1, 1)

    if method == 'multiclass':
        y_train = onehot(y_train)
        y_test = onehot(y_test)

    inp_layer =  [et[0] for et in embeddings_tensors]
    inp_layer += [ct[0] for ct in continuous_tensors]
    inp_embed =  [et[1] for et in embeddings_tensors]
    inp_embed += [ct[1] for ct in continuous_tensors]


    activation, loss, metrics = ('sigmoid', 'binary_crossentropy', 'accuracy')
    if metrics:
        metrics = [metrics]

    # d = merge(inp_embed, mode='concat')
    d = concatenate(inp_embed)
    d = Flatten()(d)
    # 2_. layer to normalise continous columns with the embeddings
    d = BatchNormalization()(d)
    d = Dense(100, activation='relu', kernel_regularizer=l1_l2(l1=0.01, l2=0.01))(d)
    # d = Dropout(0.5)(d) # Dropout don't seem to help in this model
    d = Dense(50, activation='relu')(d)
    # d = Dropout(0.5)(d) # Dropout don't seem to help in this model
    d = Dense(y_train.shape[1], activation=activation)(d)
    deep = Model(inp_layer, d)

    print(deep.summary())
    plot_model(deep, to_file='model.png', show_shapes=True)

    deep.compile(Adam(0.01), loss=loss, metrics=metrics)

    plot_model(deep, to_file='model_deep.png', show_shapes=True)

    deep.fit(X_train, y_train, batch_size=64, nb_epoch=10)
    results = deep.evaluate(X_test, y_test)

    print( "\n", results)


deep(df_train, df_test, embedding_cols, cont_cols, target, model_type, method)

```

### 3 Wide&Depth模型

#### 1 模型结构

![model.png](https://upload-images.jianshu.io/upload_images/14744153-62d800ac7605f52e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2 源码解析

```
method = 'logistic'
model_type = 'wide_depth'
train_data
test_data
```

### 4 训练数据

age| workclass| fnlwgt| education| education_num|marital_status| occupation| relationship| race| gender|capital_gain| capital_loss|hours_per_week| native_country|income_bracket
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
39| State-gov| 77516| Bachelors| 13| Never-married| Adm-clerical| Not-in-family| White| Male| 2174| 0| 40| United-States| <=50K
50| Self-emp-not-inc| 83311| Bachelors| 13| Married-civ-spouse| Exec-managerial| Husband| White| Male| 0| 0| 13| United-States| <=50K



