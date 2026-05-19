# Titantic
MLW
Spaceship Titanic 预测项目说明

项目简介
本项目用于解决 Kaggle 竞赛“Spaceship Titanic”中的乘客运输状态预测问题。通过特征工程、缺失值处理和 CatBoost 分类器，对测试集中的乘客是否被运输进行预测。

文件结构
- train.csv : 训练数据，包含标签列 Transported
- test.csv  : 测试数据，不含标签
- submission.csv : 最终预测结果，符合竞赛提交格式
- Untitled-2.ipynb : 完整代码实现

依赖库
Python 3 环境下需要安装：
- numpy
- pandas
- scikit-learn
- catboost
安装命令：pip install numpy pandas scikit-learn catboost

数据处理与特征工程步骤

1. 数据加载与合并
   - 加载 train.csv 和 test.csv
   - 合并两个数据集，便于统一进行特征工程和缺失值填补

2. 异常值截断
   - 对消费金额特征（RoomService、FoodCourt、ShoppingMall、Spa、VRDeck）的极端值进行截断

3. 从原始字段提取新特征
   - 从 PassengerId 提取 TeamId 和 Teamcode
   - 从 Cabin 提取 Deck（甲板）、Num（编号）、Side（左右舷）
   - 将 CryoSleep 和 VIP 转换为 0/1 整型
   - 计算 GroupSize（同一团队人数）和 FamilySize（同一姓氏人数）
   - 创建 Is_Solo 表示是否独行

4. 消费相关逻辑填补
   - 若 CryoSleep = 1，则所有消费金额设为 0
   - 若有任何消费金额 > 0，则强制将 CryoSleep 设为 0
   - 计算 Total_Spend 总消费额

5. 缺失值填补（基于业务规则）
   - VIP：根据年龄、居住星球、目的地等条件填补
   - HomePlanet：利用团队编号和姓氏进行前向/后向填充，再根据甲板信息补充
   - Age：根据 VIP、是否有消费等条件使用中位数填补
   - Num：按 TeamId 组内中位数填补
   - Destination：按姓氏众数填补，剩余缺失值用全局众数

6. 比率特征（基于训练标签）
   - 计算每个 Deck 和 Side 在训练集中的运输比例，作为特征映射到全体数据

7. 编码与迭代填补
   - 删除无用列：PassengerId、Cabin、Name、Lastname
   - 对类别特征（HomePlanet、Destination、Deck、Side）进行独热编码
   - 使用 IterativeImputer 迭代填补剩余的少量缺失值

8. 特征分组与标准化
   - 对 Age、Total_Spend、Num 进行分箱，生成分组特征
   - 对消费金额特征应用 log1p 变换
   - 使用 StandardScaler 对所有特征进行标准化
   - 最后删除 TeamId 和 Teamcode

最终特征列表
以下特征用于模型训练：
Age, VIP, RoomService, Spa, VRDeck, Is_Solo, Total_Spend,
Deck_trans_ratio, Side_trans_ratio,
HomePlanet_Earth, HomePlanet_Mars,
Destination_55 Cancri e, Destination_TRAPPIST-1e,
Deck_C, Deck_E, Deck_T,
Age_group, Total_Spend_Group, Num_group

模型与训练
- 模型：CatBoostClassifier
- 参数：
    depth = 4
    iterations = 3000
    learning_rate = 0.01
    thread_count = -1（使用全部 CPU 核心）
- 评估：5折交叉验证
- 输出：预测结果保存为 submission.csv

运行方法
1. 将 train.csv 和 test.csv 放在与脚本相同的目录下
2. 运行 Jupyter Notebook 中的所有代码单元格，或将代码保存为 .py 文件执行
3. 生成的 submission.csv 可直接提交到 Kaggle

输出示例
交叉验证准确率: mean = 0.7985, std = 0.0032

特征重要性示例：
Feature Importance
0   Total_Spend         34.12
1   VRDeck               8.45
...

注意事项
- 需要 scikit-learn 版本 >= 0.21（因为使用了 experimental 中的 IterativeImputer）
- IterativeImputer 运行时间较长，建议在性能较好的机器上运行
