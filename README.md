# Leandojo: Theorem Proving with Retrieval-Augmented Language Mode
# 发表
作者：Kaiyu Yang, Aidan M.Swope, Alex Gu, Rahul Chalamala, Peiyang Song, Shixing Yu, Saad Godil, Ryan Prenger, Anima Anandkumar  
学校：Caltech（加州理工学院）, NVIDIA（英伟达）, MIT（麻省理工学院）, UC Santa Barbara（加州大学圣塔芭芭拉分校）, UT Austin（得克萨斯大学奥斯汀分校）  
在 NeurIPS 2023 (神经信息处理系统大会) 发表
# 背景
推理是AI的根本目标，一个突出的任务为自动定理证明（ATP）：自动生成用形式逻辑表示的定理的证明 —— 在很多应用中是不切实际的  
-->  交互式定理证明（ITP）：人类与证明助手交互来构建证明  
-->  机器学习可以实现ITP的自动化：模型与证明助手进行交互  
现有的基于LLM的证明器都不是开源的，他们使用私有的预训练数据且计算需求十分庞大，这使得分布式训练和与证明助手交互不能完全重现 ，给用于定理证明的机器学习方法的研究带来了障碍  
关注的是在数学家中流行的证明助手Lean  
证明时，策略（证明树中的边）可以带参数，可以组合，也可以用户来新定义，这使得定理证明对机器学习具有挑战性；其次，生成策略时使用的前提来自Lean中的数学库（含有数十万个定义和定理），这导致很难选择正确的前提
# 要解决的问题
1.Lean中的数学库包含人类编写的定理和证明的源代码，但是，由于源代码缺少人类在使用Lean时可以访问的运行时信息（如：证明步骤（策略）之间的证明状态），它不适合训练证明器  
2.前提选择：现有的基于LLM的证明器生成下一个证明步骤（策略）时仅将当前状态作为输入，然而，证明定理很依赖数学库中的前提，但将所有可能的前提都纳入LLM的输入太大了 ，且现有的方法不能推广到真正新颖的场景  
3.与Lean进行可靠交互  
4.随机分割定理可能会高估证明器的性能，它允许证明器通过记忆来证明许多定理  
# 如何解决
1.Leandojo提取了以下在源代码中不可直接见的信息：文件依赖关系，抽象语法树，策略及其前后的状态，前提的定义和使用位置并修改Lean来记录这些信息，修改后的Lean仅用于数据提取  
2.引入ReProver：使用检索来显式地选择前提    
给定当前的证明状态，它检索少量可能有用的前提，并在状态和检索的前提的连接上生成策略。在证明时，该模型在每一步都会生成多个策略候选项，这些候选项被运用于最佳优先搜索算法以寻找证明  
该检索器建立在DRP的基础上并有两个算法创新：（1）在证明定理时不是所有前提都是可访问的，Leandojo对Lean代码执行程序分析确认可访问的前提，将检索限制在可访问的前提中，包括定理之前在同一文件中定义的前提和从其他文件导入的前提（2）提出In-file negatives： a simple mechanism to find hard negatives in premise selection, which samples negative premises defined in the same Lean source file as the ground truth premise.（？）  
3.以编程方式与Lean进行交互：Leandojo将Lean变成一个gym-like的环境，在这个环境中，证明器可以观察证明状态，运行策略来改变状态，并接收错误或证明完成的反馈  
4.构建了一个用于前提选择和定理证明的基准--Leandojo基准（由Lean的数学库中的98734个定理和证明组成），并创建了一个具有挑战性的数据分割--novel_premises，要求证明器推广到依赖于从未在训练中使用过的新前提的定理    
# 实验
在Leandojo 基准上对ReProver进行评估  
1.前提选择  
在Leandojo基准中选择至少有一个前提的策略，使用策略前的状态来检索100个前提，并计算信息检索中的标准指标 R@k（检索到的前k个前提的召回率）和MRR（平均倒数秩）
分别与经典BM25，从所有前提中检索以及没有In-file negatives 进行对比，ReProver的两项指标均高于以上三种
