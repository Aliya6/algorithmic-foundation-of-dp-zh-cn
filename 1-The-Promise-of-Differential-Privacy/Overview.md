# 一、差分隐私的承诺

**差分隐私** 描述了数据持有者对数据主体的承诺：“无论您将数据用于任何研究或分析，都不会受到不利影响或其他影响。” 差分数据库机制可以使机密数据广泛用于准确的数据分析，而无需诉诸数据清洗，数据使用协议，数据保护计划。 ，或其他受限方面。但是，保证隐私性的同时，将消耗数据实用性：《信息恢复基本法》指出，对太多问题的过于准确的回答将以一种惊人的方式破坏隐私。关于差异隐私的算法研究的目标是将这种不可避免性推迟尽可能长的时间。   

差异隐私解决了一个悖论，即在学习有关人口的有用信息的同时，对个人一无所知。  

医学数据库可能会告诉我们，吸烟会导致癌症，影响保险公司对吸烟者长期医疗费用的看法。吸烟者受到分析的伤害了吗？如果保险公司知道他吸烟，他的保险费可能会上涨。他可能也会得到帮助。但保险公司学习他的健康风险，使他进入戒烟计划。吸烟者的隐私被侵犯了吗？当然，研究结束后对他的了解比以前更多，但他的信息是不是“泄露”了？差异隐私将认为它不是，理由是对吸烟者的影响是相同的独立于他是否在研究中。是这项研究得出的结论影响了吸烟者，而不是他在数据集中的存在与否影响了实验得出的结论。  

差异隐私保证了相同的结论，例如，吸烟会导致癌症，这与是否有人选择进入或退出数据集无关。具体地说，它确保任何输出序列（对查询的响应）在“本质上”发生的可能性相同，与任何个体的存在或不存在无关。这里，概率被隐私机制（由数据持有者控制）所做的随机选择所取代，术语“本质上”被抽象为参数 ε。较小的 ε 将产生更好的隐私（和更不准确的响应）。  

差异隐私是一个定义，而不是一个算法。对于给定的计算任务 T 和给定的 ε 值，将有许多不同的私有算法以 **ε-差异隐私** 方式实现 T。有些算法会比其他算法更准确。当 ε 很小时，很难为任务 T 找到一个高精度的ε-差分隐私算法，就像为一个特定的计算任务找到一个数值稳定的算法一样。  