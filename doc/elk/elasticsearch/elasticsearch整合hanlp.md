<!-- TOC -->

- [下载与配置](#下载与配置)
    - [方式一、Maven](#方式一maven)
    - [方式二、下载jar、data、hanlp.properties](#方式二下载jardatahanlpproperties)
        - [1、下载：[data.zip](http://nlp.hankcs.com/download.php?file=data)](#1下载dataziphttpnlphankcscomdownloadphpfiledata)
        - [2、下载jar和配置文件：[hanlp-release.zip](http://nlp.hankcs.com/download.php?file=jar)](#2下载jar和配置文件hanlp-releaseziphttpnlphankcscomdownloadphpfilejar)
- [调用方法](#调用方法)
    - [1. 第一个Demo](#1-第一个demo)
    - [2. 标准分词](#2-标准分词)
    - [3. NLP分词](#3-nlp分词)
    - [4. 索引分词](#4-索引分词)
    - [5. N-最短路径分词](#5-n-最短路径分词)
    - [6. CRF分词](#6-crf分词)
    - [7. 极速词典分词](#7-极速词典分词)
    - [8. 用户自定义词典](#8-用户自定义词典)
    - [9. 中国人名识别](#9-中国人名识别)
    - [10. 音译人名识别](#10-音译人名识别)
    - [11. 日本人名识别](#11-日本人名识别)
    - [12. 地名识别](#12-地名识别)
    - [13. 机构名识别](#13-机构名识别)
    - [14. 关键词提取](#14-关键词提取)
    - [15. 自动摘要](#15-自动摘要)
    - [16. 拼音转换](#16-拼音转换)
    - [17. 简繁转换](#17-简繁转换)
    - [18. 文本推荐](#18-文本推荐)

<!-- /TOC -->
HanLP: Han Language Processing
=====

HanLP提供下列功能：

* 中文分词
    * HMM-Bigram（速度与精度最佳平衡；一百兆内存）
        * [最短路分词](https://github.com/hankcs/HanLP#1-%E7%AC%AC%E4%B8%80%E4%B8%AAdemo)、[N-最短路分词](https://github.com/hankcs/HanLP#5-n-%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84%E5%88%86%E8%AF%8D)
    * 由字构词（侧重精度，全世界最大语料库，可识别新词；适合NLP任务）
        * [感知机分词](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)、[CRF分词](https://github.com/hankcs/HanLP#6-crf%E5%88%86%E8%AF%8D)
    * 词典分词（侧重速度，每秒数千万字符；省内存）
        * [极速词典分词](https://github.com/hankcs/HanLP#7-%E6%9E%81%E9%80%9F%E8%AF%8D%E5%85%B8%E5%88%86%E8%AF%8D)
    * 所有分词器都支持：
        * [索引全切分模式](https://github.com/hankcs/HanLP#4-%E7%B4%A2%E5%BC%95%E5%88%86%E8%AF%8D)
        * [用户自定义词典](https://github.com/hankcs/HanLP#8-%E7%94%A8%E6%88%B7%E8%87%AA%E5%AE%9A%E4%B9%89%E8%AF%8D%E5%85%B8)
        * [兼容繁体中文](https://github.com/hankcs/HanLP/blob/master/src/test/java/com/hankcs/demo/DemoPerceptronLexicalAnalyzer.java#L29)
        * [训练用户自己的领域模型](https://github.com/hankcs/HanLP/wiki)
* 词性标注
    * [HMM词性标注](https://github.com/hankcs/HanLP/blob/master/src/main/java/com/hankcs/hanlp/seg/Segment.java#L584)（速度快）
    * [感知机词性标注](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)、[CRF词性标注](https://github.com/hankcs/HanLP/wiki/CRF%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90)（精度高）
* 命名实体识别
    * 基于HMM角色标注的命名实体识别 （速度快）
        * [中国人名识别](https://github.com/hankcs/HanLP#9-%E4%B8%AD%E5%9B%BD%E4%BA%BA%E5%90%8D%E8%AF%86%E5%88%AB)、[音译人名识别](https://github.com/hankcs/HanLP#10-%E9%9F%B3%E8%AF%91%E4%BA%BA%E5%90%8D%E8%AF%86%E5%88%AB)、[日本人名识别](https://github.com/hankcs/HanLP#11-%E6%97%A5%E6%9C%AC%E4%BA%BA%E5%90%8D%E8%AF%86%E5%88%AB)、[地名识别](https://github.com/hankcs/HanLP#12-%E5%9C%B0%E5%90%8D%E8%AF%86%E5%88%AB)、[实体机构名识别](https://github.com/hankcs/HanLP#13-%E6%9C%BA%E6%9E%84%E5%90%8D%E8%AF%86%E5%88%AB)
    * 基于线性模型的命名实体识别（精度高）
        * [感知机命名实体识别](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)、[CRF命名实体识别](https://github.com/hankcs/HanLP/wiki/CRF%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90)
* 关键词提取
    * [TextRank关键词提取](https://github.com/hankcs/HanLP#14-%E5%85%B3%E9%94%AE%E8%AF%8D%E6%8F%90%E5%8F%96)
* 自动摘要
    * [TextRank自动摘要](https://github.com/hankcs/HanLP#15-%E8%87%AA%E5%8A%A8%E6%91%98%E8%A6%81)
* [拼音转换](https://github.com/hankcs/HanLP#17-%E6%8B%BC%E9%9F%B3%E8%BD%AC%E6%8D%A2)
    * 多音字、声母、韵母、声调
* [简繁转换](https://github.com/hankcs/HanLP#18-%E7%AE%80%E7%B9%81%E8%BD%AC%E6%8D%A2)
    * 简繁分歧词（简体、繁体、臺灣正體、香港繁體）
* [文本推荐](https://github.com/hankcs/HanLP#19-%E6%96%87%E6%9C%AC%E6%8E%A8%E8%8D%90)
    * 语义推荐、拼音推荐、字词推荐
* [文本分类](https://github.com/hankcs/HanLP/wiki/%E6%96%87%E6%9C%AC%E5%88%86%E7%B1%BB%E4%B8%8E%E6%83%85%E6%84%9F%E5%88%86%E6%9E%90)
    * [情感分析](https://github.com/hankcs/HanLP/wiki/%E6%96%87%E6%9C%AC%E5%88%86%E7%B1%BB%E4%B8%8E%E6%83%85%E6%84%9F%E5%88%86%E6%9E%90#%E6%83%85%E6%84%9F%E5%88%86%E6%9E%90)
* [文本聚类](https://github.com/hankcs/HanLP/wiki/%E6%96%87%E6%9C%AC%E8%81%9A%E7%B1%BB)
    - KMeans、Repeated Bisection、自动推断聚类数目k
* [语料库工具](https://github.com/hankcs/HanLP/tree/master/src/main/java/com/hankcs/hanlp/corpus)
    - 部分默认模型训练自小型语料库，鼓励用户自行训练。所有模块提供[训练接口](https://github.com/hankcs/HanLP/wiki)，语料可参考[98年人民日报语料库](http://file.hankcs.com/corpus/pku98.zip)。

------

## 下载与配置

### 方式一、Maven

为了方便用户，特提供内置了数据包的Portable版，只需在pom.xml加入：

```xml
<dependency>
    <groupId>com.hankcs</groupId>
    <artifactId>hanlp</artifactId>
    <version>portable-1.7.5</version>
</dependency>
```

零配置，即可使用基本功能（除由字构词、依存句法分析外的全部功能）。如果用户有自定义的需求，可以参考方式二，使用hanlp.properties进行配置（Portable版同样支持hanlp.properties）。

### 方式二、下载jar、data、hanlp.properties

HanLP将数据与程序分离，给予用户自定义的自由。

#### 1、下载：[data.zip](http://nlp.hankcs.com/download.php?file=data) 

下载后解压到任意目录，接下来通过配置文件告诉HanLP数据包的位置。

HanLP中的数据分为*词典*和*模型*，其中*词典*是词法分析必需的，*模型*是句法分析必需的。

    data
    │
    ├─dictionary
    └─model

用户可以自行增删替换，如果不需要句法分析等功能的话，随时可以删除model文件夹。

- 模型跟词典没有绝对的区别，隐马模型被做成人人都可以编辑的词典形式，不代表它不是模型。
- GitHub代码库中已经包含了data.zip中的词典，直接编译运行自动缓存即可；模型则需要额外下载。

#### 2、下载jar和配置文件：[hanlp-release.zip](http://nlp.hankcs.com/download.php?file=jar)

配置文件的作用是告诉HanLP数据包的位置，只需修改第一行

    root=D:/JavaProjects/HanLP/

为data的**父目录**即可，比如data目录是`/Users/hankcs/Documents/data`，那么`root=/Users/hankcs/Documents/` 。

最后将`hanlp.properties`放入classpath即可，对于多数项目，都可以放到src或resources目录下，编译时IDE会自动将其复制到classpath中。除了配置文件外，还可以使用环境变量`HANLP_ROOT`来设置`root`。安卓项目请参考[demo](https://github.com/hankcs/HanLPAndroidDemo)。

如果放置不当，HanLP会提示当前环境下的合适路径，并且尝试从项目根目录读取数据集。

## 调用方法

HanLP几乎所有的功能都可以通过工具类`HanLP`快捷调用，当你想不起来调用方法时，只需键入`HanLP.`，IDE应当会给出提示，并展示HanLP完善的文档。

所有Demo都位于[com.hankcs.demo](https://github.com/hankcs/HanLP/tree/master/src/test/java/com/hankcs/demo)下，比文档覆盖了更多细节，更新更及时，**强烈建议运行一遍**。此处仅列举部分常用接口。

### 1. 第一个Demo

```java
System.out.println(HanLP.segment("你好，欢迎使用HanLP汉语处理包！"));
```
- 内存要求
  * 内存120MB以上（-Xms120m -Xmx120m -Xmn64m），标准数据包（35万核心词库+默认用户词典），分词测试正常。全部词典和模型都是惰性加载的，不使用的模型相当于不存在，可以自由删除。
  * HanLP对词典的数据结构进行了长期的优化，可以应对绝大多数场景。哪怕HanLP的词典上百兆也无需担心，因为在内存中被精心压缩过。如果内存非常有限，请使用小词典。HanLP默认使用大词典，同时提供小词典，请参考配置文件章节。
- 写给正在编译HanLP的开发者
  * 如果你正在编译运行从Github检出的HanLP代码，并且没有下载data缓存，那么首次加载词典/模型会发生一个*自动缓存*的过程。
  * *自动缓存*的目的是为了加速词典载入速度，在下次载入时，缓存的词典文件会带来毫秒级的加载速度。由于词典体积很大，*自动缓存*会耗费一些时间，请耐心等待。
  * *自动缓存*缓存的不是明文词典，而是双数组Trie树、DAWG、AhoCorasickDoubleArrayTrie等数据结构。

### 2. 标准分词

```java
List<Term> termList = StandardTokenizer.segment("商品和服务");
System.out.println(termList);
```
- 说明
  * HanLP中有一系列“开箱即用”的静态分词器，以`Tokenizer`结尾，在接下来的例子中会继续介绍。
  * `HanLP.segment`其实是对`StandardTokenizer.segment`的包装。
  * 分词结果包含词性，每个词性的意思请查阅[《HanLP词性标注集》](http://www.hankcs.com/nlp/part-of-speech-tagging.html#h2-8)。
- 算法详解
  * [《词图的生成》](http://www.hankcs.com/nlp/segment/the-word-graph-is-generated.html)

### 3. NLP分词

```java
System.out.println(NLPTokenizer.segment("我新造一个词叫幻想乡你能识别并标注正确词性吗？"));
// 注意观察下面两个“希望”的词性、两个“晚霞”的词性
System.out.println(NLPTokenizer.analyze("我的希望是希望张晚霞的背影被晚霞映红").translateLabels());
System.out.println(NLPTokenizer.analyze("支援臺灣正體香港繁體：微软公司於1975年由比爾·蓋茲和保羅·艾倫創立。"));
```
- 说明
  * NLP分词`NLPTokenizer`会执行词性标注和命名实体识别，由[结构化感知机序列标注框架](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)支撑。
  * 默认模型训练自`9970`万字的大型综合语料库，是已知范围内**全世界最大**的中文分词语料库。语料库规模决定实际效果，面向生产环境的语料库应当在千万字量级。欢迎用户在自己的语料上[训练新模型](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)以适应新领域、识别新的命名实体。

### 4. 索引分词

```java
List<Term> termList = IndexTokenizer.segment("主副食品");
for (Term term : termList)
{
    System.out.println(term + " [" + term.offset + ":" + (term.offset + term.word.length()) + "]");
}
```
- 说明
  * 索引分词`IndexTokenizer`是面向搜索引擎的分词器，能够对长词全切分，另外通过`term.offset`可以获取单词在文本中的偏移量。
  * 任何分词器都可以通过基类`Segment`的`enableIndexMode`方法激活索引模式。

### 5. N-最短路径分词

```java
Segment nShortSegment = new NShortSegment().enableCustomDictionary(false).enablePlaceRecognize(true).enableOrganizationRecognize(true);
Segment shortestSegment = new DijkstraSegment().enableCustomDictionary(false).enablePlaceRecognize(true).enableOrganizationRecognize(true);
String[] testCase = new String[]{
        "今天，刘志军案的关键人物,山西女商人丁书苗在市二中院出庭受审。",
        "刘喜杰石国祥会见吴亚琴先进事迹报告团成员",
        };
for (String sentence : testCase)
{
    System.out.println("N-最短分词：" + nShortSegment.seg(sentence) + "\n最短路分词：" + shortestSegment.seg(sentence));
}
```
- 说明
  * N最短路分词器`NShortSegment`比最短路分词器慢，但是效果稍微好一些，对命名实体识别能力更强。
  * 一般场景下最短路分词的精度已经足够，而且速度比N最短路分词器快几倍，请酌情选择。
- 算法详解
  * [《N最短路径的Java实现与分词应用》](http://www.hankcs.com/nlp/segment/n-shortest-path-to-the-java-implementation-and-application-segmentation.html)

### 6. CRF分词

```java
        CRFLexicalAnalyzer analyzer = new CRFLexicalAnalyzer();
        String[] tests = new String[]{
            "商品和服务",
            "上海华安工业（集团）公司董事长谭旭光和秘书胡花蕊来到美国纽约现代艺术博物馆参观",
            "微软公司於1975年由比爾·蓋茲和保羅·艾倫創立，18年啟動以智慧雲端、前端為導向的大改組。" // 支持繁体中文
        };
        for (String sentence : tests)
        {
            System.out.println(analyzer.analyze(sentence));
        }
```
- 说明
  * CRF对新词有很好的识别能力，但是开销较大。
- 算法详解
  * [《CRF中文分词、词性标注与命名实体识别》](https://github.com/hankcs/HanLP/wiki/CRF%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90)

### 7. 极速词典分词

```java
/**
 * 演示极速分词，基于AhoCorasickDoubleArrayTrie实现的词典分词，适用于“高吞吐量”“精度一般”的场合
 * @author hankcs
 */
public class DemoHighSpeedSegment
{
    public static void main(String[] args)
    {
        String text = "江西鄱阳湖干枯，中国最大淡水湖变成大草原";
        System.out.println(SpeedTokenizer.segment(text));
        long start = System.currentTimeMillis();
        int pressure = 1000000;
        for (int i = 0; i < pressure; ++i)
        {
            SpeedTokenizer.segment(text);
        }
        double costTime = (System.currentTimeMillis() - start) / (double)1000;
        System.out.printf("分词速度：%.2f字每秒", text.length() * pressure / costTime);
    }
}
```
- 说明
  * 极速分词是词典最长分词，速度极其快，精度一般。
  * 在i7-6700K上跑出了4500万字每秒的速度。
- 算法详解
  * [《Aho Corasick自动机结合DoubleArrayTrie极速多模式匹配》](http://www.hankcs.com/program/algorithm/aho-corasick-double-array-trie.html)

### 8. 用户自定义词典

```java
/**
 * 演示用户词典的动态增删
 *
 * @author hankcs
 */
public class DemoCustomDictionary
{
    public static void main(String[] args)
    {
        // 动态增加
        CustomDictionary.add("攻城狮");
        // 强行插入
        CustomDictionary.insert("白富美", "nz 1024");
        // 删除词语（注释掉试试）
//        CustomDictionary.remove("攻城狮");
        System.out.println(CustomDictionary.add("单身狗", "nz 1024 n 1"));
        System.out.println(CustomDictionary.get("单身狗"));

        String text = "攻城狮逆袭单身狗，迎娶白富美，走上人生巅峰";  // 怎么可能噗哈哈！

        // AhoCorasickDoubleArrayTrie自动机扫描文本中出现的自定义词语
        final char[] charArray = text.toCharArray();
        CustomDictionary.parseText(charArray, new AhoCorasickDoubleArrayTrie.IHit<CoreDictionary.Attribute>()
        {
            @Override
            public void hit(int begin, int end, CoreDictionary.Attribute value)
            {
                System.out.printf("[%d:%d]=%s %s\n", begin, end, new String(charArray, begin, end - begin), value);
            }
        });

        // 自定义词典在所有分词器中都有效
        System.out.println(HanLP.segment(text));
    }
}
```
- 说明
  * `CustomDictionary`是一份全局的用户自定义词典，可以随时增删，影响全部分词器。另外可以在任何分词器中关闭它。通过代码动态增删不会保存到词典文件。
  * 中文分词≠词典，词典无法解决中文分词，`Segment`提供高低优先级应对不同场景，请参考[FAQ](https://github.com/hankcs/HanLP/wiki/FAQ#%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BF%AE%E6%94%B9%E4%BA%86%E8%AF%8D%E5%85%B8%E8%BF%98%E6%98%AF%E6%B2%A1%E6%9C%89%E6%95%88%E6%9E%9C)。
- 追加词典
  * `CustomDictionary`主词典文本路径是`data/dictionary/custom/CustomDictionary.txt`，用户可以在此增加自己的词语（不推荐）；也可以单独新建一个文本文件，通过配置文件`CustomDictionaryPath=data/dictionary/custom/CustomDictionary.txt; 我的词典.txt;`来追加词典（推荐）。
  * 始终建议将相同词性的词语放到同一个词典文件里，便于维护和分享。
- 词典格式
  * 每一行代表一个单词，格式遵从`[单词] [词性A] [A的频次] [词性B] [B的频次] ...` 如果不填词性则表示采用词典的默认词性。
  * 词典的默认词性默认是名词n，可以通过配置文件修改：`全国地名大全.txt ns;`如果词典路径后面空格紧接着词性，则该词典默认是该词性。
  * 在统计分词中，并不保证自定义词典中的词一定被切分出来。用户可在理解后果的情况下通过`Segment#enableCustomDictionaryForcing`强制生效。
  * 关于用户词典的更多信息请参考**词典说明**一章。
- 算法详解
  * [《Trie树分词》](http://www.hankcs.com/program/java/tire-tree-participle.html)
  * [《Aho Corasick自动机结合DoubleArrayTrie极速多模式匹配》](http://www.hankcs.com/program/algorithm/aho-corasick-double-array-trie.html)

### 9. 中国人名识别

```java
String[] testCase = new String[]{
        "签约仪式前，秦光荣、李纪恒、仇和等一同会见了参加签约的企业家。",
        "王国强、高峰、汪洋、张朝阳光着头、韩寒、小四",
        "张浩和胡健康复员回家了",
        "王总和小丽结婚了",
        "编剧邵钧林和稽道青说",
        "这里有关天培的有关事迹",
        "龚学平等领导,邓颖超生前",
        };
Segment segment = HanLP.newSegment().enableNameRecognize(true);
for (String sentence : testCase)
{
    List<Term> termList = segment.seg(sentence);
    System.out.println(termList);
}
```
- 说明
  * 目前分词器基本上都默认开启了中国人名识别，比如`HanLP.segment()`接口中使用的分词器等等，用户不必手动开启；上面的代码只是为了强调。
  * 有一定的误命中率，比如误命中`关键年`，则可以通过在`data/dictionary/person/nr.txt`加入一条`关键年 A 1`来排除`关键年`作为人名的可能性，也可以将`关键年`作为新词登记到自定义词典中。
  * 如果你通过上述办法解决了问题，欢迎向我提交pull request，词典也是宝贵的财富。
  * 建议NLP用户使用感知机或CRF词法分析器，精度更高。
- 算法详解
  * [《实战HMM-Viterbi角色标注中国人名识别》](http://www.hankcs.com/nlp/chinese-name-recognition-in-actual-hmm-viterbi-role-labeling.html)

### 10. 音译人名识别

```java
String[] testCase = new String[]{
                "一桶冰水当头倒下，微软的比尔盖茨、Facebook的扎克伯格跟桑德博格、亚马逊的贝索斯、苹果的库克全都不惜湿身入镜，这些硅谷的科技人，飞蛾扑火似地牺牲演出，其实全为了慈善。",
                "世界上最长的姓名是简森·乔伊·亚历山大·比基·卡利斯勒·达夫·埃利奥特·福克斯·伊维鲁莫·马尔尼·梅尔斯·帕特森·汤普森·华莱士·普雷斯顿。",
        };
Segment segment = HanLP.newSegment().enableTranslatedNameRecognize(true);
for (String sentence : testCase)
{
    List<Term> termList = segment.seg(sentence);
    System.out.println(termList);
}
```
- 说明
  * 目前分词器基本上都默认开启了音译人名识别，用户不必手动开启；上面的代码只是为了强调。
- 算法详解
  * [《层叠隐马模型下的音译人名和日本人名识别》](http://www.hankcs.com/nlp/name-transliteration-cascaded-hidden-markov-model-and-japanese-personal-names-recognition.html)

### 11. 日本人名识别

```java
String[] testCase = new String[]{
        "北川景子参演了林诣彬导演的《速度与激情3》",
        "林志玲亮相网友:确定不是波多野结衣？",
};
Segment segment = HanLP.newSegment().enableJapaneseNameRecognize(true);
for (String sentence : testCase)
{
    List<Term> termList = segment.seg(sentence);
    System.out.println(termList);
}
```
- 说明
  * 目前标准分词器默认关闭了日本人名识别，用户需要手动开启；这是因为日本人名的出现频率较低，但是又消耗性能。
- 算法详解
  * [《层叠隐马模型下的音译人名和日本人名识别》](http://www.hankcs.com/nlp/name-transliteration-cascaded-hidden-markov-model-and-japanese-personal-names-recognition.html)

### 12. 地名识别

```java
String[] testCase = new String[]{
        "武胜县新学乡政府大楼门前锣鼓喧天",
        "蓝翔给宁夏固原市彭阳县红河镇黑牛沟村捐赠了挖掘机",
};
Segment segment = HanLP.newSegment().enablePlaceRecognize(true);
for (String sentence : testCase)
{
    List<Term> termList = segment.seg(sentence);
    System.out.println(termList);
}
```
- 说明
  * 目前标准分词器都默认关闭了地名识别，用户需要手动开启；这是因为消耗性能，其实多数地名都收录在核心词典和用户自定义词典中。
  * 在生产环境中，能靠词典解决的问题就靠词典解决，这是最高效稳定的方法。
  * 建议对命名实体识别要求较高的用户使用[感知机词法分析器](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)。
- 算法详解
  * [《实战HMM-Viterbi角色标注地名识别》](http://www.hankcs.com/nlp/ner/place-names-to-identify-actual-hmm-viterbi-role-labeling.html)

### 13. 机构名识别

```java
String[] testCase = new String[]{
    "我在上海林原科技有限公司兼职工作，",
    "我经常在台川喜宴餐厅吃饭，",
    "偶尔去地中海影城看电影。",
};
Segment segment = HanLP.newSegment().enableOrganizationRecognize(true);
for (String sentence : testCase)
{
    List<Term> termList = segment.seg(sentence);
    System.out.println(termList);
}
```
- 说明
  * 目前分词器默认关闭了机构名识别，用户需要手动开启；这是因为消耗性能，其实常用机构名都收录在核心词典和用户自定义词典中。
  * HanLP的目的不是演示动态识别，在生产环境中，能靠词典解决的问题就靠词典解决，这是最高效稳定的方法。
  * 建议对命名实体识别要求较高的用户使用[感知机词法分析器](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)。
- 算法详解
  * [《层叠HMM-Viterbi角色标注模型下的机构名识别》](http://www.hankcs.com/nlp/ner/place-name-recognition-model-of-the-stacked-hmm-viterbi-role-labeling.html)

### 14. 关键词提取

```java
String content = "程序员(英文Programmer)是从事程序开发、维护的专业人员。一般将程序员分为程序设计人员和程序编码人员，但两者的界限并不非常清楚，特别是在中国。软件从业人员分为初级程序员、高级程序员、系统分析员和项目经理四大类。";
List<String> keywordList = HanLP.extractKeyword(content, 5);
System.out.println(keywordList);
```
- 说明
  * 内部采用`TextRankKeyword`实现，用户可以直接调用`TextRankKeyword.getKeywordList(document, size)`
- 算法详解
  * [《TextRank算法提取关键词的Java实现》](http://www.hankcs.com/nlp/textrank-algorithm-to-extract-the-keywords-java-implementation.html)

### 15. 自动摘要

```java
String document = "算法可大致分为基本算法、数据结构的算法、数论算法、计算几何的算法、图的算法、动态规划以及数值分析、加密算法、排序算法、检索算法、随机化算法、并行算法、厄米变形模型、随机森林算法。\n" +
        "算法可以宽泛的分为三类，\n" +
        "一，有限的确定性算法，这类算法在有限的一段时间内终止。他们可能要花很长时间来执行指定的任务，但仍将在一定的时间内终止。这类算法得出的结果常取决于输入值。\n" +
        "二，有限的非确定算法，这类算法在有限的时间内终止。然而，对于一个（或一些）给定的数值，算法的结果并不是唯一的或确定的。\n" +
        "三，无限的算法，是那些由于没有定义终止定义条件，或定义的条件无法由输入的数据满足而不终止运行的算法。通常，无限算法的产生是由于未能确定的定义终止条件。";
List<String> sentenceList = HanLP.extractSummary(document, 3);
System.out.println(sentenceList);
```
- 说明
  * 内部采用`TextRankSentence`实现，用户可以直接调用`TextRankSentence.getTopSentenceList(document, size)`。
- 算法详解
  * [《TextRank算法自动摘要的Java实现》](http://www.hankcs.com/nlp/textrank-algorithm-java-implementation-of-automatic-abstract.html)

### 16. 拼音转换

```java
/**
 * 汉字转拼音
 * @author hankcs
 */
public class DemoPinyin
{
    public static void main(String[] args)
    {
        String text = "重载不是重任";
        List<Pinyin> pinyinList = HanLP.convertToPinyinList(text);
        System.out.print("原文,");
        for (char c : text.toCharArray())
        {
            System.out.printf("%c,", c);
        }
        System.out.println();

        System.out.print("拼音（数字音调）,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin);
        }
        System.out.println();

        System.out.print("拼音（符号音调）,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getPinyinWithToneMark());
        }
        System.out.println();

        System.out.print("拼音（无音调）,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getPinyinWithoutTone());
        }
        System.out.println();

        System.out.print("声调,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getTone());
        }
        System.out.println();

        System.out.print("声母,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getShengmu());
        }
        System.out.println();

        System.out.print("韵母,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getYunmu());
        }
        System.out.println();

        System.out.print("输入法头,");
        for (Pinyin pinyin : pinyinList)
        {
            System.out.printf("%s,", pinyin.getHead());
        }
        System.out.println();
    }
}
```
- 说明
  * HanLP不仅支持基础的汉字转拼音，还支持声母、韵母、音调、音标和输入法首字母首声母功能。
  * HanLP能够识别多音字，也能给繁体中文注拼音。
  * 最重要的是，HanLP采用的模式匹配升级到`AhoCorasickDoubleArrayTrie`，性能大幅提升，能够提供毫秒级的响应速度！
- 算法详解
  * [《汉字转拼音与简繁转换的Java实现》](http://www.hankcs.com/nlp/java-chinese-characters-to-pinyin-and-simplified-conversion-realization.html#h2-17)

### 17. 简繁转换

```java
/**
 * 简繁转换
 * @author hankcs
 */
public class DemoTraditionalChinese2SimplifiedChinese
{
    public static void main(String[] args)
    {
        System.out.println(HanLP.convertToTraditionalChinese("用笔记本电脑写程序"));
        System.out.println(HanLP.convertToSimplifiedChinese("「以後等妳當上皇后，就能買士多啤梨慶祝了」"));
    }
}
```
- 说明
  * HanLP能够识别简繁分歧词，比如`打印机=印表機`。许多简繁转换工具不能区分“以后”“皇后”中的两个“后”字，HanLP可以。
- 算法详解
  * [《汉字转拼音与简繁转换的Java实现》](http://www.hankcs.com/nlp/java-chinese-characters-to-pinyin-and-simplified-conversion-realization.html#h2-17)

### 18. 文本推荐

```java
/**
 * 文本推荐(句子级别，从一系列句子中挑出与输入句子最相似的那一个)
 * @author hankcs
 */
public class DemoSuggester
{
    public static void main(String[] args)
    {
        Suggester suggester = new Suggester();
        String[] titleArray =
        (
                "威廉王子发表演说 呼吁保护野生动物\n" +
                "《时代》年度人物最终入围名单出炉 普京马云入选\n" +
                "“黑格比”横扫菲：菲吸取“海燕”经验及早疏散\n" +
                "日本保密法将正式生效 日媒指其损害国民知情权\n" +
                "英报告说空气污染带来“公共健康危机”"
        ).split("\\n");
        for (String title : titleArray)
        {
            suggester.addSentence(title);
        }

        System.out.println(suggester.suggest("发言", 1));       // 语义
        System.out.println(suggester.suggest("危机公共", 1));   // 字符
        System.out.println(suggester.suggest("mayun", 1));      // 拼音
    }
}
```
- 说明
  * 在搜索引擎的输入框中，用户输入一个词，搜索引擎会联想出最合适的搜索词，HanLP实现了类似的功能。
  * 可以动态调节每种识别器的权重
