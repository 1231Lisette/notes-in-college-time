# diffusion policy

一个好的robot learning方法可以：

1. 不需要i工程洞察和设计能力，直接从数据中进行学习
2. 方法可以产出某种结构化的动作，以执行各种复杂的任务
3. 最好还能有一些理论基础和数学分析手段，而不是一个纯纯的黑盒



1. diffusion：适合建模多模态动作分布
2. GAN：常用于模仿学习，GAIL
3. VAE：支持随机策略生成
4. Transformer：适合长时程任务
5. Flow-based Models：通过逆变换建模动作分布