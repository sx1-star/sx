# Leandojo:  Theorem Proving with Retrieval-Augmented Language Mode
# 发表
作者：Kaiyu Yang, Aidan M.Swope, Alex Gu, Rahul Chalamala, Peiyang Song, Shixing Yu, Saad Godil, Ryan Prenger, Anima Anandkumar  
学校：Caltech（加州理工学院）, NVIDIA（英伟达）, MIT（麻省理工学院）, UC Santa Barbara（加州大学圣塔芭芭拉分校）, UT Austin（得克萨斯大学奥斯汀分校）  
在 NeurIPS 2023 (神经信息处理系统大会) 发表
# 背景
推理是AI的根本目标，一个突出的任务为自动定理证明（ATP）：自动生成用形式逻辑表示的定理的证明 —— 在很多应用中是不切实际的  
-->  交互式定理证明（ITP）：人类与证明助手交互来构建证明  
-->  机器学习可以实现ITP的自动化：模型与证明助手进行交互  
（现有的基于LLM的证明器都不是开源的），他们使用私有的预训练数据且计算需求十分庞大，这使得分布式训练和与证明助手交互不能完全重现 ，给用于定理证明的机器学习方法的研究带来了障碍  
LISA：基于ISAbelle的环境，从ISAbelle中提取引理和定理作为语料库，并在上面进行训练    
PACT：为提高策略预测性能，从证明项中提取大量数据与通常的策略预测目标一起进行联合训练    
HTPS：在线训练，开发了方程式环境，创建了一个由随机生成的合成定理及其证明组成的训练集  
Thor：提出hammer方法进行前提选择  
>现有的能与证明助手交互的环境（或可提高交互性能的方法所用的训练数据集未开源）  

关注的是在数学家中流行的证明助手Lean
1.提供一种统一的语言  
2.提供一种策略系统  
证明时，策略（证明树中的边）可以带参数，可以组合，也可以用户来新定义，这使得定理证明对机器学习具有挑战性；其次，生成策略时使用的前提来自Lean中的数学库（含有数十万个定义和定理），这导致很难选择正确的前提  

# 验证
欧几里得算法：  
设 a 和 b 是两个非零自然数，且 a≥b  
重复以下步骤，直到 b=0 ：  
    1）.计算 r=a mod b（即 a 除以 b 的余数）  
    2）.令 a=b, b=r  
当 b=0 时，a 就是 gcd(a,b)  

# 要解决的问题
1.Lean中的数学库包含人类编写的定理和证明的源代码，但是，由于源代码缺少人类在使用Lean时可以访问的运行时信息（如：证明步骤（策略）之间的证明状态），它不适合训练证明器  
![2c951af9d90110e8462f95e516c350d](https://github.com/user-attachments/assets/5f3ca55c-eac3-4533-ac9c-d4c566cc31ea)  
如图，抽象语法树等均不可见  
抽象语法树常用来表示前提和结论，可以通过遍历和操作这些树来执行逻辑推理，从而生成证明步骤（还有检错等作用）  
2.前提选择：现有的基于LLM的证明器生成下一个证明步骤（策略）时仅将当前状态作为输入，然而，证明定理很依赖数学库中的前提，但将所有可能的前提都纳入LLM的输入太大了 ，且现有的方法不能推广到真正新颖的场景   
3.与Lean进行可靠交互  
4.随机分割定理可能会高估证明器的性能，它允许证明器通过记忆来证明许多定理    
随机分割定理：在机器学习中将数据集随机分割成训练集、验证集和测试集的过程  
验证集：用于模型的选择和调优  
（1）模型选择：在训练过程中，可以尝试不同的模型架构，然后在验证集上评估它们的性能，选择表现最好的模型。  
（2）防止过拟合：在验证集上监控模型的性能，可以及时发现模型是否开始过拟合训练数据，及时避免（过拟合：模型对训练数据的适应性过强，失去了泛化能力）  
（3）模型参数调优：根据验证集的性能，可以调整模型参数，提升模型的性能  
证明器（模型）的记忆：模型从训练数据中学习到的知识，这些知识以模型参数的形式存储  
# 如何解决
1.Leandojo提取了以下在源代码中不可直接见的信息：文件依赖关系，抽象语法树，策略及其前后的状态，前提的定义位置和使用位置。  
文件依赖关系：生成一个有向无环图，节点是文件，边是文件之间的导入关系  
lean具有导出依赖项、ast、状态和策略的基本支持，但无法解析前提的全称（e.g.mod_self & Nat.mod_self）并找到它们的定义，因此修改Lean来记录这些信息  
2.前提选择 -- 开发ReProver：使用检索来显式地选择前提    
给定当前的证明状态，它检索少量可能有用的前提，并在检索到的前提上生成策略。    
该模型在每一步都会生成多个策略候选项，这些候选项被运用于最佳优先搜索算法以寻找证明  

该检索器建立在DPR的基础上并有两个算法创新  
DPR：用密集向量表示状态和候选前提库，通过向量相似度来检索最相关的前提，DPR的性能主要取决于负例（Negative example）的质量  
负例：不有助于证明目标定理的前提    
（1）在证明定理时不是所有前提都是可访问的，Leandojo将检索限制在可访问的前提中，包括在要证明的定理之前在同一文件中定义的前提和从其他文件导入的前提     
（2）提出In-file negatives来训练模型：选取文件内否定和随机否定进行训练  
随机否定：在训练数据中随机选择一些前提作为负例   
文件内否定：一种在前提选择中找到难负例的简单机制， 对在同一Lean源文件中定义的与真实前提相关的负例进行采样   
难负例：难以与正例区分的不相关前提   
3.与Lean进行可靠交互 -- 以编程方式与lean进行交互：Leandojo将Lean变成一个gym-like的环境：在这个环境中，证明器可以观察证明状态，运行策略来改变状态，并接收错误或证明完成的反馈  
initialize(theorem) : 给定要证明的定理，Leandojo返回初始状态  
run_tac(state,tactical) : 在给定状态上运行一个策略，并返回下一个状态，若策略执行不成功，则返回错误状态  
4.构建了一个用于前提选择和定理证明的基准--Leandojo基准（由Lean的数学库中的98734个定理和证明组成），并创建了一个具有挑战性的数据分割--novel_premises，要求证明器推广到依赖于从未在训练中使用过的新前提的定理  
# 实验
在 Leandojo基准 上对ReProver进行评估  
1.前提选择  
在Leandojo基准中选择至少有一个前提的策略，使用策略前的状态作为查询来检索100个前提，并计算信息检索中的标准指标 R@k（检索到的前k个前提的召回率）和MRR（平均倒数秩）
将经典BM25作为基线，比较了两种消融：  
（1）从所有前提中检索而不仅仅是可访问的前提  
（2）没有 in-file negatives   
消融：通过移除模型或系统中的某些组件、特征或步骤来分析它们对整体性能的贡献  
BM25：评估文档与查询之间的相关性，通过计算查询词在文档中的出现频率和文档的长度来估计，不是一个机器学习算法，基于概率检索框架，不需要复杂的模型训练  
![9d6bef56318c0139e22fe871b85a259](https://github.com/user-attachments/assets/17b58e0b-49c1-4565-9f96-cb80e7c9c6f0)

2.定理证明  
