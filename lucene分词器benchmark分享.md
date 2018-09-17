# lucene分词器benchmark分享

标签：分词

------

## 调研目的

- 海量数据搜索的需求

- 分词是搜索的重要环节--客户短信

## lucene自带分词器缺点
分词器过于简单，产生索引过多，存在大量无意义的垃圾分词，无法满足现实的需求。（索引，语义）

## 常用分词器分类

### 1.单纯的词典匹配方法：Ik分词器,mmseg
  
  **优点**：速度快，实现简单。
  
  **缺点**: 对歧义和未登录词处理效果不好。

### 2.词典+统计+机器学习:Jieba,Ansj,Jcseg

对语料库进行词频和词性的统计，采用了HMM、CRF等序列标注模型训练

**优点**：分词效果好，功能多

**缺点**: 处理速度慢

Jieba分词器：基于词频的最大概率切分组合；对于未登录词，采用了HMM算法。

Ansj分词器(n-Gram+CRF+HMM)

Jcseg(基于MMSEG改进，多种模式)

## 分词效果举例

**原始字符串:**

>工信处女干事每月经过下属科室都要亲口交代24口交换机等技术性器件的安装工作

***Standard分词器***：
>工|信|处|女|干|事|每|月|经|过|下|属|科|室|都|要|亲|口|交|代|24|口|交|换|机|等|技|术|性|器|件|的|安|装|工|作|

***Cjk分词器***：
>工信|信处|处女|女干|干事|事每|每月|月经|经过|过下|下属|属科|科室|室都|都要|要亲|亲口|口交|交代|24|口交|交换|换机|机等|等技|技术|术性|性器|器件|件的|的安|安装|装工|工作|

***Ngram分词器***：
>工|工信|信|信处|处|处女|女|女干|干|干事|事|事每|每|每月|月|月经|经|经过|过|过下|下|下属|属|属科|科|科室|室|室都|都|都要|要|要亲|亲|亲口|口|口交|交|交代|代|代2|2|24|4|4口|口|口交|交|交换|换|换机|机|机等|等|等技|技|技术|术|术性|性|性器|器|器件|件|件的|的|的安|安|安装|装|装工|工|工作|作|

***Ik_max_word***：
>工|信|处女|干事|每月|月经|经过|下属|科室|都要|亲口|口交|交代|24|口交|口|交换机|交换|换机|等|技术性|技术|性器|器件|的|安装|装工|工作|

***Ik_smart***：(消除歧义)
>工|信|处女|干事|每月|经过|下属|科室|都要|亲口|交代|24|口交|换机|等|技术性|器件|的|安装|工作|

***Jieba分词器***：
>工信处|干事|女干事|每月|经过|下属|科室|都|要|亲口|交代|24|口|交换|换机|交换机|等|技术|技术性|器件|的|安装|工作|

***Ansj(base_ansj)***：
>工|信|处|女|干事|每月|经过|下属|科室|都|要|亲口|交代|24|口|交换机|等|技术性|器件|的|安装|工作|

***Ansj(dic_ansj)***：
>工信处|女|干事|每月|经过|下属|科室|都|要|亲口|交代|24|口|交换机|等|技术性|器件|的|安装|工作|

***Ansj(index_ansj)***：
>工信处|工|信|处女|处|女干事|女|干事|干|事|每月|每|月经|月|经过|经|过|下属|下|属|科室|科|室|都|要|亲口|亲|口交|口|交代|交|代|24口|口交|口|交换机|交换|交|换机|换|机|等|技术性|技术|技|术|性器|性|器件|器|件|的|安装工|安装|安|装工|装|工作|工|作|

***Ansj(nlp_ansj)***：
>工信|处女|干事|每月|经过|下属|科室|都|要|亲口|交代|24口|交换机|等|技术性|器件|的|安装|工作

***Ansj(query_ansj)***：
>工信处|女干事|每月|经过|下属|科室|都|要|亲口|交代|24口|交换机|等|技术性|器件|的|安装|工作|

***Jcseg(COMPLEX_MODE)***：
>工信处|女|干事|每月|经过|下属|科室|都|要|亲口|交代|24|口|交换机|等|技术性|器件|的|安装|工作|

***Jcseg(DETECT_MODE)***:
>工信处|女|干事|每月|经过|下属|科室|都|要|亲口|交代|口交|换|机|等|技术性|器件|的|安装|工作|

***Jcseg(SEARCH_MODE)***：
>工|工信处|信|处|处女|女|干|干事|事|每|每月|月|月经|经|经过|过|下|下属|属|科|科室|室|都|都|要|要|亲|亲口|口|口交|交|交代|代|24|口|口交|交|交换|交换机|换|机|等|等|技|技术|技术性|术|性|性器|器|器件|件|的|的|安|安装|装|工|工作|作|

***Jcseg(SIMPLE_MODE)***：
>工信处|女|干事|每月|经过|下属|科室|都|要|亲口|交代|24|口交|换|机|等|技术性|器件|的|安装|工作|

***Jcseg(NLP_MODE)***：
>工信处|女|干事|每月|经过|下属|科室|都|要|亲口|交代|24|口|交换机|等|技术性|器件|的|安装|工作|

## 分词器IK介绍（细粒度分词，智能分词）
![Ik_smart处理流程][2]
**Ik_smart消除歧义策略:**
![Ik_smart消除歧义策略][1]
**Ik_smart处理流程举例:**
![Ik_smart处理流程举例][3]

## Jieba分词器介绍
（有lucene插件和低版本的es插件，还没找到适合5.x的es插件）
![Jieba基本处理流程][4]

![Jieba举例DAG][5]

![此处输入图片的描述][6]

## 分词处理性能
表格
<table>
 <tr>
  <td>
   分词器名称
  </td>
  <td>
 <b>堆内内存</b>
  </td>
  <td>
   输入文件大小
  </td>
  <td>
   读入文件时间
  </td>
  <td>
   分词处理时间
  </td>
  <td>
   写入文件时间
  </td>
  <td>
   输出文件大小
  </td>
  <td>
   <b>数据膨胀率</b>
  </td>
  <td>
   <b>分词处理速度</b>
  </td>
 </tr>
 <tr>
  <td>
   Standard分词器
  </td>
  <td>
   1.78M
  </td>
  <td>
   3.9G
  </td>
  <td>
   30s
  </td>
  <td>
   215s
  </td>
  <td>
   45s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   18,000K/s
  </td>
 </tr>
 <tr>
  <td>
   CJK分词器
  </td>
  <td>
   6.80M
  </td>
  <td>
   3.9G
  </td>
  <td>
   29s
  </td>
  <td>
   277s
  </td>
  <td>
   64s
  </td>
  <td>
   5.92G
  </td>
  <td>
   1.54
  </td>
  <td>
   14,873K/s
  </td>
 </tr>
 <tr>
  <td>
   Ngram分词器
  </td>
  <td>
   --
  </td>
  <td>
   3.9G
  </td>
  <td>
   30s
  </td>
  <td>
   1400s
  </td>
  <td>
   155s
  </td>
  <td>
   11.7G
  </td>
  <td>
   3
  </td>
  <td>
   2,786K/s
  </td>
 </tr>
 <tr>
  <td>
   Ik_max_word
  </td>
  <td>
   9.69M
  </td>
  <td>
   3.9G
  </td>
  <td>
   30s
  </td>
  <td>
   800s
  </td>
  <td>
   60s
  </td>
  <td>
   5.8G
  </td>
  <td>
   1.5
  </td>
  <td>
   4,875K/s
  </td>
 </tr>
 <tr>
  <td>
   Ik_smart
  </td>
  <td>
   <b>2.86M</b>
  </td>
  <td>
   3.9G
  </td>
  <td>
   28s
  </td>
  <td>
   689s
  </td>
  <td>
   23s
  </td>
  <td>
   3.38G
  </td>
  <td>
   0.88
  </td>
  <td>
   <b>5,660K/s</b>
  </td>
 </tr>
 <tr>
  <td>
   Jieba分词器
  </td>
  <td>
   11.22M
  </td>
  <td>
   3.9G
  </td>
  <td>
   33s
  </td>
  <td>
   1376s
  </td>
  <td>
   31s
  </td>
  <td>
   4.04G
  </td>
  <td>
   1.05
  </td>
  <td>
   3,000K/s
  </td>
 </tr>
 <tr>
  <td>
   Ansj(base_ansj)
  </td>
  <td>
   9.0M
  </td>
  <td>
   3.9G
  </td>
  <td>
   32s
  </td>
  <td>
   742s
  </td>
  <td>
   40s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   <b>5,500K/s</b>
  </td>
 </tr>
 <tr>
  <td>
   Ansj( index_ansj)
  </td>
  <td>
   <b>1.71M</b>
  </td>
  <td>
   3.9G
  </td>
  <td>
   32s
  </td>
  <td>
   2511s
  </td>
  <td>
   58s
  </td>
  <td>
   7.0G
  </td>
  <td>
   1.8
  </td>
  <td>
   1,645K/s
  </td>
 </tr>
 <tr>
  <td>
   Ansj(query_ansj)
  </td>
  <td>
   <b>1.72M</b>
  </td>
  <td>
   3.9G
  </td>
  <td>
   33s
  </td>
  <td>
   1490s
  </td>
  <td>
   30s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   2,617K/s
  </td>
 </tr>
 <tr>
  <td>
   Ansj(dic_ansj)
  </td>
  <td>
   9.54M
  </td>
  <td>
   3.9G
  </td>
  <td>
   29s
  </td>
  <td>
   1493s
  </td>
  <td>
   28s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   2,764K/s
  </td>
 </tr>
 <tr>
  <td>
   Ansj(nlp_ansj)
  </td>
  <td>
   13.24M
  </td>
  <td>
   3.9G
  </td>
  <td>
   30s
  </td>
  <td>
   5536s
  </td>
  <td>
   32s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   746K/s
  </td>
 </tr>
 <tr>
  <td>
   Jcseg(SEARCH_MODE)
  </td>
  <td>
   9.42M
  </td>
  <td>
   3.9G
  </td>
  <td>
   32s
  </td>
  <td>
   1252s
  </td>
  <td>
   50s
  </td>
  <td>
   7.7G
  </td>
  <td>
   1.75
  </td>
  <td>
   3,115K/s
  </td>
 </tr>
 <tr>
  <td>
   Jcseg(SIMPLE_MODE)
  </td>
  <td>
   9.40M
  </td>
  <td>
   3.9G
  </td>
  <td>
   33s
  </td>
  <td>
   743s
  </td>
  <td>
   33s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   <b>5,248K/s</b>
  </td>
 </tr>
 <tr>
  <td>
   Jcseg(COMPLEX_MODE)
  </td>
  <td>
   9.29M
  </td>
  <td>
   3.9G
  </td>
  <td>
   33s
  </td>
  <td>
   1680s
  </td>
  <td>
   28s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   2,321K/s
  </td>
 </tr>
 <tr>
  <td>
   Jcseg(DETECT_MODE)
  </td>
  <td>
   <b>448K</b>
  </td>
  <td>
   3.9G
  </td>
  <td>
   30s
  </td>
  <td>
   1054s
  </td>
  <td>
   23s
  </td>
  <td>
   3.0G
  </td>
  <td>
   0.76
  </td>
  <td>
   3,700K/S
  </td>
 </tr>
 <tr>
  <td>
   Jcseg(NLP_MODE)
  </td>
  <td>
   9.43M
  </td>
  <td>
   3.9G
  </td>
  <td>
   33s
  </td>
  <td>
   1568s
  </td>
  <td>
   35s
  </td>
  <td>
   3.9G
  </td>
  <td>
   1
  </td>
  <td>
   2,632K/s
  </td>
 </tr>
 <tr>
  <td>
   
  </td>
 </tr>
</table>

## 分词效果--黄金标准(精准分词)

<table class="table table-striped table-condensed">
   <tr>
      <th>
         ----------
      </th>
      <th colspan="3">pku数据集（中文简体）</th> 
      <th colspan="3">msr数据集（中文简体）</th>
      <th colspan="3">cityu数据集（中文繁体）</th>
   </tr>
   <tr>
        <td>分词器名称</td>
  <td>召回率</td>
  <td>准确率</td>
  <td><b>F值</b></td>
  <td>召回率</td>
  <td>准确率</td>
  <td><b>F值</b></td>
  <td>召回率</td>
  <td>准确率</td>
  <td>F值</td>
 </tr>
 <tr>
  <td>Standard分词器</td>
  <td>0.353</td>
  <td>0.205</td>
  <td>0.259</td>
  <td>0.351</td>
  <td>0.194</td>
  <td>0.25</td>
  <td>0.383</td>
  <td>0.221</td>
  <td>0.28</td>
 </tr>
 <tr>
  <td>CJK分词器</td>
  <td>0.558</td>
  <td>0.331</td>
  <td>0.415</td>
  <td>0.558</td>
  <td>0.316</td>
  <td>0.404</td>
  <td>0.538</td>
  <td>0.322</td>
  <td>0.403</td>
 </tr>
 <tr>
  <td>Ngram分词器</td>
  <td>0.913</td>
  <td>0.26</td>
  <td>0.405</td>
  <td>0.911</td>
  <td>0.252</td>
  <td>0.395</td>
  <td>0.9</td>
  <td>0.256</td>
  <td>0.398</td>
 </tr>
 <tr>
  <td>IkMaxWord:停词词典为空</td>
  <td>0.858</td>
  <td>0.505</td>
  <td>0.636</td>
  <td>0.865</td>
  <td>0.495</td>
  <td>0.63</td>
  <td>0.542</td>
  <td>0.326</td>
  <td>0.407</td>
 </tr>
 <tr>
  <td>IkMaxWord:停词非空</td>
  <td>0.758</td>
  <td>0.475</td>
  <td>0.584</td>
  <td>0.756</td>
  <td>0.463</td>
  <td>0.575</td>
  <td>0.458</td>
  <td>0.291</td>
  <td>0.356</td>
 </tr>
 <tr>
  <td>IK-Smart:停词为空</td>
  <td>0.654</td>
  <td>0.753</td>
  <td>0.7</td>
  <td>0.684</td>
  <td>0.753</td>
  <td>0.717</td>
  <td>0.477</td>
  <td>0.343</td>
  <td>0.399</td>
 </tr>
 <tr>
  <td>IK-Smart:停词非空</td>
  <td>0.557</td>
  <td>0.722</td>
  <td>0.629</td>
  <td>0.578</td>
  <td>0.721</td>
  <td>0.642</td>
  <td>0.394</td>
  <td>0.302</td>
  <td>0.342</td>
 </tr>
 <tr>
  <td>Jieba分词器</td>
  <td>0.843</td>
  <td>0.747</td>
  <td>0.792</td>
  <td>0.873</td>
  <td>0.75</td>
  <td>0.807</td>
  <td>0.683</td>
  <td>0.679</td>
  <td>0.679</td>
 </tr>
 <tr>
  <td>Ansj(base_ansj)</td>
  <td>0.916</td>
  <td>0.87</td>
  <td><b>0.892</b></td>
  <td>0.893</td>
  <td>0.83</td>
  <td><b>0.86</b></td>
  <td>0.557</td>
  <td>0.373</td>
  <td>0.447</td>
 </tr>
 <tr>
  <td>Ansj( index_ansj)</td>
  <td>0.976</td>
  <td>0.367</td>
  <td>0.533</td>
  <td>0.981</td>
  <td>0.352</td>
  <td>0.518</td>
  <td>0.609</td>
  <td>0.29</td>
  <td>0.393</td>
 </tr>
 <tr>
  <td>Ansj(dic_ansj)</td>
  <td>0.761</td>
  <td>0.827</td>
  <td><b>0.793</b></td>
  <td>0.813</td>
  <td>0.852</td>
  <td><b>0.832</b></td>
  <td>0.532</td>
  <td>0.375</td>
  <td>0.44</td>
 </tr>
 <tr>
  <td>Ansj(nlp_ansj)</td>
  <td>0.739</td>
  <td>0.855</td>
  <td><b>0.793</b></td>
  <td>0.792</td>
  <td>0.867</td>
  <td><b>0.828</b></td>
  <td>0.661</td>
  <td>0.683</td>
  <td>0.672</td>
 </tr>
 <tr>
  <td>Ansj(query_ansj)</td>
  <td>0.76</td>
  <td>0.858</td>
  <td><b>0.806</b></td>
  <td>0.806</td>
  <td>0.861</td>
  <td><b>0.832</b></td>
  <td>0.527</td>
  <td>0.383</td>
  <td>0.444</td>
 </tr>
 <tr>
  <td>Jcseg(COMPLEX_MODE)</td>
  <td>0.753</td>
  <td>0.806</td>
  <td>0.779</td>
  <td>0.8</td>
  <td>0.822</td>
  <td>0.811</td>
  <td>0.511</td>
  <td>0.361</td>
  <td>0.423</td>
 </tr>
 <tr>
  <td>Jcseg(DETECT_MODE)</td>
  <td>0.729</td>
  <td>0.793</td>
  <td>0.759</td>
  <td>0.786</td>
  <td>0.807</td>
  <td>0.797</td>
  <td>-</td>
  <td>-</td>
  <td>-</td>
 </tr>
 <tr>
  <td>Jcseg(NLP_MODE)</td>
  <td>0.749</td>
  <td>0.809</td>
  <td>0.778</td>
  <td>0.795</td>
  <td>0.825</td>
  <td>0.81</td>
  <td>0.503</td>
  <td>0.358</td>
  <td>0.418</td>
 </tr>
 <tr>
  <td>Jcseg(SEARCH_MODE)</td>
  <td>0.962</td>
  <td>0.339</td>
  <td>0.501</td>
  <td>0.967</td>
  <td>0.32</td>
  <td>0.481</td>
  <td>0.593</td>
  <td>0.228</td>
  <td>0.329</td>
 </tr>
 <tr>
  <td>Jcseg(SIMPLE_MODE)</td>
  <td>0.751</td>
  <td>0.795</td>
  <td>0.772</td>
  <td>0.787</td>
  <td>0.801</td>
  <td>0.794</td>
  <td>0.506</td>
  <td>0.356</td>
  <td>0.418</td>
   </tr>
</table>

## 分词总结

分词效果（F值由高到低）：base_Ansj > query_Ansj > nlp/dic_Ansj.

分词处理性能排序（由快到慢）：Ik_smart > base_Ansj >Jcseg_SIMPLE_MODE.

分词占用堆内内存（由少到多）： Jcseg_DETECT_MODE > (base_Ansj/query_Ansj) > Ik_smart 

支持繁体功能的是：Jieba，nlp_Ansj

扩展功能较多的是：Ansj（5种模式）和Jcseg（6种模式）

### **ik_smart**

**适用场景**:只需配置词语。适合于速度快，对准确率要求不太高的场合

**优点**：性能全面优于ik_max_word。数据膨胀率小，占用堆内内存少，处理速度快

**缺点**：只是简单的根据配置好的词库进行分词，依赖于词库是否完善。消除歧义方面获得的不一定是最准确的效果

### **jieba**

**适用场景**: 

***支持三种分词模式***

a. 精确模式，试图将句子最精确地切开，适合文本分析；

b. 全模式，把句子中所有的可以成词的词语都扫描出来,        速度非常快，但是不能解决歧义； 

c. 搜索引擎模式，在精确模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。

***支持繁体分词***

***支持自定义词典***

**优点**：准确率比ik_smart高

**缺点**：占用内存多，处理速度一般。现有的jieba插件不支持es高版本

### **ansj**

**适用场景**：目前实现了中文分词，中文姓名识别，地名识别，用户自定义词典,关键字提取，自动摘要，关键字标记等多种功能。
         ansj_index索引模式不会出现词元交叉（是在精准模式的基础上进行细切分）

**优点**：有五种模式，满足不同业务需求，其中ansj_base等部分模式适合于速度较快的场合。ansj对索引和最大颗粒分割都照顾得很到位。

**缺点**:ansj_nlp分词器性能不稳定，测试中发现处理速度极其慢，所得到的分词准确率也不一定高，鸡肋。
ansj_base的分词处理性能接近ik_smart，分词效果极高，但是占用内存高于ik_smart。

### **Jcseg**

**适用场景**：jcseg分词器使用的是mmseg算法和fmm算法，分词器类似ansj智能、人性化。停词过滤，自动词性标注，自动实体的识别。

**优点**：超越了mmseg4j，功能丰富,智能分词模式做的不错。

自动实体的识别，默认支持：电子邮件，网址，大陆手机号码，地名，人名，货币等；词库中可以自定义各种实体并且再切分中返回。

(1).简易模式：FMM算法(正向最大匹配)，适合速度要求场合。

(2).复杂模式：MMSEG四种过滤算法，具有较高的歧义去除

(3).检测模式：只返回词库中已有的词条，很适合某些应用场合。

(4).检索模式：细粒度切分，专为检索而生，除了中文处理外（不具备中文的人名，数字识别等智能功能）其他与复杂模式一致（英文，组合词等）。

(5).分隔符模式：按照给定的字符切分词条，默认是空格，特定场合的应用。

(6).NLP模式：继承自复杂模式，更改了数字，单位等词条的组合方式，增加电子邮件，大陆手机号码，网址，人名，地名，货币等以及无限种自定义实体的识别与返回。

**缺点**：追求分词效果越高的同时，处理速度越慢，占用内存越多。

简单模式分词精度做得比较差，不如Ansj的简单模式分词。（二者处理性能方面差不多）





  [1]: http://p6jdfz9vx.bkt.clouddn.com/IKsmartepl3.png
  [2]: http://p6jdfz9vx.bkt.clouddn.com/IKsmartepl6.png
  [3]: http://p6jdfz9vx.bkt.clouddn.com/IKsmartepl5.png
  [4]: http://p6jdfz9vx.bkt.clouddn.com/IKsmartepl9.png
  [5]: http://p6jdfz9vx.bkt.clouddn.com/IKsmartepl8.png
  [6]: http://p6jdfz9vx.bkt.clouddn.com/IKsmartepl10.png
