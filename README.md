# Leandojo: Theorem Proving with Retrieval-Augmented Language Mode
# 发表
作者：Kaiyu Yang, Aidan M.Swope, Alex Gu, Rahul Chalamala, Peiyang Song, Shixing Yu, Saad Godil, Ryan Prenger, Anima Anandkumar  
学校：Caltech（加州理工学院）, NVIDIA（英伟达）, MIT（麻省理工学院）, UC Santa Barbara（加州大学圣塔芭芭拉分校）, UT Austin（得克萨斯大学奥斯汀分校）  
在 NeurIPS 2023 (神经信息处理系统大会) 发表
# 背景
推理是AI的根本目标，一个突出的任务为自动定理证明（ATP）：自动生成用形式逻辑表示的定理的证明 —— 在很多应用中是不切实际的  
-->  交互式定理证明（ITP）：人类与证明助手交互来构建证明  
-->  机器学习可以实现ITP的自动化：模型与证明助手进行交互  
关注的是在数学家中流行的证明助手Lean  
证明时，策略（证明树中的边）可以带参数，可以组合，也可以用户来新定义，这使得定理证明对机器学习具有挑战性；其次，生成策略时使用的前提来自Lean中的数学库（含有数十万个定义和定理），这导致很难选择正确的前提
# 解决的问题
1.现有的基于LLM的证明器都不是开源的，他们使用私有的预训练数据且计算需求十分庞大，这使得分布式训练和与证明助手交互这两种机器学习定理证明的方法不能完全重现  
2.前提选择  
3.以编程方式与Lean进行交互  
4.随机分割定理可能会高估证明器的性能
