下游交付包：训练/测试划分数据

概述
----
本数据包用于二分类任务：预测样本是否属于“高健康风险（HighRisk）”。

本方案的原则是：
- 尽量保持“特征为 Raw”，不在上游做缺失值填补、类别编码、缩放等，以便下游根据模型自由选择预处理策略。
- 提供一套“常规异常值处理”的可选版本：仅对数值型特征做 IQR 裁剪（winsorize），并严格避免数据泄露。

数据来源与处理摘要
----------------
- 数据来自：alzheimer_s_disease_and_healthy_aging_data.csv
- 上游仅做构造标签所需的最小清洗：
  - 将用于计算标签的数值字段（如 Data_Value）强制转为数值，无法解析的置为 NaN
  - Data_Value 缺失的行无法生成标签，因此删除

标签定义
--------
- 标签列：HighRisk（0/1）
- HighRisk 的具体判定规则见 notebook 中“构造标签/Label Construction”代码段。

训练/测试划分
------------
- 采用分层抽样（stratified split），使训练集与测试集中 HighRisk 的比例尽量一致
- 随机种子由 notebook 中 RANDOM_STATE 控制，以保证可复现

异常值处理（可选，数值列）
------------------------
我们提供两套数据：raw 与 clipped。

clipped 版本对“数值型特征列”做 IQR 裁剪（winsorize）：
- 对每个数值列，在训练集上计算 Q1、Q3、IQR = Q3 - Q1
- 下界 = Q1 - k * IQR
- 上界 = Q3 + k * IQR
- 将训练集与测试集中该列的数值裁剪到 [下界, 上界] 范围内

重要：避免数据泄露
----------------
- IQR 的阈值（Q1/Q3/IQR/上下界）只在训练集上计算（fit on train）
- 测试集只复用训练集阈值进行裁剪（transform on test）
- 不使用全量数据统计量来决定异常值边界

输出文件
--------
- train_raw.csv：原始特征 + HighRisk 标签
- test_raw.csv：原始特征 + HighRisk 标签
- train_clipped.csv：数值特征做 IQR 裁剪后的版本 + HighRisk 标签
- test_clipped.csv：数值特征做 IQR 裁剪后的版本 + HighRisk 标签

下游仍需自行决定的预处理
----------------------
根据下游模型的需要，仍可能需要：
- 缺失值填补（imputation）
- 类别变量编码（categorical encoding）
- 缩放/归一化（scaling/normalization，依赖模型类型）
