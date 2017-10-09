---
title: familia小白使用
categories: [familia]
tags: [familia, 机器学习]
---

### familia到底是什么
[Familia](https://github.com/baidu/Familia)百度开源的文档主题推断工具、语义匹配计算工具。

主题模型在工业界的应用范式可以抽象为两大类: 语义表示和语义匹配。

- 语义表示 (Semantic Representation)

   对文档进行主题降维，获得文档的语义表示，这些语义表示可以应用于文本分类、文本内容分析、CTR预估等下游应用。

- 语义匹配 (Semantic Matching)

  计算文本间的语义匹配度，我们提供两种文本类型的相似度计算方式:

  - 短文本-长文本相似度计算，使用场景包括文档关键词抽取、计算搜索引擎查询和网页的相似度等等。
  - 长文本-长文本相似度计算，使用场景包括计算两篇文档的相似度、计算用户画像和新闻的相似度等等。
<!--more-->
### 小白角度看familia
   这种机器学习的项目、几乎只有大公司才玩得起的, 小公司无法投入这么大的人力物力。但是这种项目的产出真的是非常的诱人, familia是一个做到开箱即用的工具, 使用者不需要花费人力物力成本去生成模型, 而是建立在现有训练好的模型之上, 直接产出结果.


### 编译测试运行
```
git clone https://github.com/baidu/Familia.git
cd familia
sh build.sh
cd model
sh download_model.sh
cd ..
sh run_doc_distance_demo.sh
```

注意事项，链接thrid_party
```
sudo vim /etc/ld.so.conf.d/familia.conf
/data/Familia/third_party/lib
:wq
sudo ldconfig
```

### 实际环境中的运用
+ 文章识别, 针对晚上爬取的良莠不齐的文章, 我们通过主题与文章的关系检查是否是符合的文章(run_query_doc_sim_demo.sh)
+ 广告识别, 论坛中的坏蛋发小广告, 我们通过一些关键字, 可以大致识别广告，然后加上一些人工识别, 效果应该还是很好的。
