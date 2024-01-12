#  FunSearch:Making new discoveries in mathematical sciences using Large Language Models

### Introduction

  Paper: [Mathematical discoveries from program search with large language models ](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/funsearch-making-new-discoveries-in-mathematical-sciences-using-large-language-models/Mathematical-discoveries-from-program-search-with-large-language-models.pdf)

  Code: [Google-Deepmind](https://github.com/google-deepmind/funsearch)

### Methods
  1. **Best-shot Prompting**: sample best performing progams and feed them back into pormpts 
  2. Only envolve the part governing the critical program logic
  3. **Island-based evolutionary**: more exploration and avoid local optima



  ![](https://secure2.wostatic.cn/static/2SHJ2c3muew6XDUbJtKqGc/image.png?auth_key=1705067222-hR2Une5gsNkrTzPxN3V8B-0-d185db40bdcaf4d594f643549e66d7ca)



#### Specification
  1. **Input**  以评价函数evaluate对具体的问题进行规范，对候选解进行打分。

      ![](https://secure2.wostatic.cn/static/gHUL1jgC1YTpGH2UEPtNWr/image.png?auth_key=1705067222-tLUXL6zUQRWbGouTVAs5np-0-fdc196fe3dd5519afb5391b712f84f30)

      ![](https://secure2.wostatic.cn/static/vvUSaznFUoFtB2C6ZnM5Mw/image.png?auth_key=1705067222-oQYGHMzUVQ915n7Z6eA6GM-0-aac7bb657a6627ab8b9c5ab109b257bb)
  2. **Solve** 以skeleton的形式编写初始解程序，并且只演变priority 或heuristic 

      具体的，我们可以看下面链接的 priority 函数迭代变化 [FunSearch](https://deepmind.google/discover/blog/funsearch-making-new-discoveries-in-mathematical-sciences-using-large-language-models/)

      skeleton的缺点在于可发现的代码空间有限

#### Pre-trained Models
  1. 这里采用的Codey（PaLM2系列），选择了更快速的推理模型，没有选择速度慢但高质量的模型
  2. 采样的量级是 $10^6$
  3. 只要在足够大的代码语料库上训练，结果对LLM推理质量高低不太敏感

#### Evaluation
  1. LLM生成代码，通过一组输入对生成的代码进行评估和打分
  2. 使用聚合函数（均值）对不同输入之间的得分进行合并得到最终得分
  3. 打分后的程序发送到Program Database中，丢弃不正确的代码（没有在规定时间内完成执行，内存限制，无效输出等一系列Error报错）

#### Program Database
  1. 保存执行正确的程序，对程序以进行采样来创建prompt
  2. 为了避免陷入局部最优解，即鼓励生成代码的多样性，采用了 Island-based evolutionary 算法
      1. 最初的情况，将初始代码放入Program中，并复制m=10份
      2. 采样一个island，在这个island中采样一个得分更高，代码长度更短的program
      3. 周期性的丢弃islands 中表现较差的一半的 islands，然后将剩余island中得分最高的program复制一份放在island中初始化来进行演化

          ![](https://secure2.wostatic.cn/static/khUKYDR7F7zTGLJJUaQE76/image.png?auth_key=1705067222-nKhY7YUAYLcm4yRu6DhFoY-0-ccbf03094923c45384b2290b95a09fd1)

#### Prompt
  1. 从单个island中采样k个program，根据得分进行排序，组合成为一个单一的prompt
  2. $v_0$后缀是最高得分program，$v_1$是次高分program，$v_2$是我们所要生成的program

      ![](https://secure2.wostatic.cn/static/hfLj8pY3C7VLJGC1faVguR/image.png?auth_key=1705067222-228Kz9yBqCtsqVPyYUcSfk-0-dcdaca2ff4af29a4f77e8abfa34bdbe4)

      ![](https://secure2.wostatic.cn/static/rFLKExQKcgqCmC26FLZRrN/image.png?auth_key=1705066075-dYeuwKBTBveiB84kFrjfdT-0-25918879d3aacafc080985cb89cdee4c)
  3. 选择k=2的原因是因为结果最好

#### Distributed approach（这里不太懂）
  1. 通过一个分布式系统实现Funsearch，包括program database，samplers，evaluators

      ![](https://secure2.wostatic.cn/static/qD7gEQYwYYighZ4zzT2UNE/image.png?auth_key=1705067222-pwFw8CACSxpUVgjMM14Ny3-0-5035150e5ec4ddfe248740f3ac010a92)
  2. 优点：
      1. LLM采样和Evaluator同时进行
      2. 对多个采样器和评估器进行拓展，评估器在CPU上进行运行

#### **Note**
  1. 在每个island里面，通过signature对program进行聚类（**Algorithm F.15**），signature指的是一系列输入通过evaluate所得到的分数集合

      ![](https://secure2.wostatic.cn/static/5o8m8f9LkqzrduRaaeQiQX/image.png?auth_key=1705066040-g8iHZvTBZEXWwDzem8kjzh-0-88f07df6d78b3b14be48d9ffcb75b286)

      ![](https://secure2.wostatic.cn/static/4VqBjgXT1JzoEJ3F6of2PV/image.png?auth_key=1705066040-nHZhQNr9FNtbhAVB39r3Fs-0-fa398dcd41a0af883c52aab9f0c0bb07)
  2. 采样得分较高的簇（**Algorithm F.13**），再从簇中采样program（**Algorithm F.16**）

      ![](https://secure2.wostatic.cn/static/vXNfHT1NTSaBwHAgUeEjJH/image.png?auth_key=1705066040-r5hazS4CKL7gq726m5wT1W-0-6949dd44b0a9c55aabd19f1602605d8e)

      ![](https://secure2.wostatic.cn/static/54tKRXCvsoVRLXd3r6iLQ8/image.png?auth_key=1705066040-enwbAEhKu54hsPgR4yBGma-0-46bc54b0871d19e91151917cefcb7d08)
  3. 超参

      ![](https://secure2.wostatic.cn/static/bb6iyS3YGsfgHG4uYQnWWr/image.png?auth_key=1705066040-b9Dnk5KAWCN5PPK896RuKe-0-850f649f9889b028f8ac14886f8ead27)



### Result
  1. **Cap Set** 找到了 n=8 目前为止最大集合512个点，与此同时，集合大小的下确界也提高到了 $2.2202^n$

      ![](https://secure2.wostatic.cn/static/cyy6C5F1enr3JeJ6jFyP1/image.png?auth_key=1705066954-4mCEZACvei91Pn8qZpGDPX-0-003d26151442a71197e248d93e1910e7)

      ![](https://secure2.wostatic.cn/static/9PYnedotoVNnGiU5mgeKL1/image.png?auth_key=1705067055-397saztkTBCMa9quMz2ybi-0-6a734fc35ed2713814e3499dd5acf940)
  2. **Bin packing **与理论下界在100k的时候只差0.03%

      ![](https://secure2.wostatic.cn/static/icZtDg6vFXuNUnKozBU4CE/image.png?auth_key=1705067155-qgHHxzHMfqUGFbS66diDtY-0-1ab826594e6bae15f8d1a498a75e3eaa)

  

### Ablation
  1. 这里的消融实验证明以下几类的有效性

      ![](https://secure2.wostatic.cn/static/fqsgeYT9rM8ZN5zMuDi2F3/image.png?auth_key=1705066652-f1DHeLjvw5JPHiJJ4g5EgJ-0-e946b2a35935008ef39a218c141ff265)

      1. 是否演化函数
      2. 用Skeleton只演化priority 和 heuristic
      3. 多样性
      4. 聚合结果 
  2. Codey的稳定性其实不如StarCoder，但Codey可以找到Cap Set 问题的最大容量，StarCoder不行

      ![](https://secure2.wostatic.cn/static/mp2GHnMomNGvLT9uWrdrjj/image.png?auth_key=1705067095-kMP2Bg3Rpby49eKZwKkDkF-0-53dcb5d06ec1993090b6890ae9fd82fd)

