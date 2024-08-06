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
一些常见的证明器参数：  
1.前提和假设：参数包括如何定义和输入这些前提。   
2.推理规则：证明器使用的推理规则是关键参数。这些规则定义了如何从前提到结论进行逻辑推导。  
3.证明风格和格式：不同的证明器可能支持不同的证明风格（如自然演绎、归谬法等）和格式（如形式化证明、自然语言证明等）。  
4. 交互级别：证明器可能支持不同级别的用户交互，从完全自动化的证明生成到用户逐步引导的交互式证明过程。  
5.性能参数：包括证明器的计算资源需求、处理时间、内存使用等。  
6. 错误处理和反馈：证明器如何处理逻辑错误、提供反馈和建议修正    
7. 知识库和数据源：证明器可能依赖于特定的知识库或数据源。参数包括如何访问和利用这些资源。    
8. 可扩展性和灵活性：证明器是否支持扩展新的推理规则、前提类型或证明风格    
