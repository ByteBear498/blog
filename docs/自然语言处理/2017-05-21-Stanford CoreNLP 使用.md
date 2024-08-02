---
date: 2017-05-21 13:18:00
status: public
title: Stanford CoreNLP 使用
Tags: 
 - 自然语言处理
 - Stanford CoreNLP
categories:
 - NLP
---
## 介绍
斯坦福CoreNLP提供了一套自然语言分析工具。斯坦福CoreNLP集成了许多斯坦福的NLP工具，包括词性（POS），命名实体识别（NER）， 语法解析，指代消解，情感分析，自举模式学习和开放信息提取工具。

如果有如下需求，要以选择Stanford CoreNLP：
* 具有良好语法分析工具的集成工具包
* 快速，可靠地分析任意文本
* 整体最高品质的文字分析
* 支持一些主要（人类）语言
* 可用于大多数主要现代编程语言的接口
* 能够作为简单的Web服务运行

Github地址：[Stanford CoreNLP GitHub site.](https://github.com/stanfordnlp/CoreNLP)

## 使用前准备
Stanford CoreNLP是基于JAVA的，最新版本需要JDK1.8。JAR包可以通用官网，或者MAVEN下载到。如果使用MAVEN的话，可以使用如下配置。
```xml
<dependencies>
<dependency>
    <groupId>edu.stanford.nlp</groupId>
    <artifactId>stanford-corenlp</artifactId>
    <version>3.7.0</version>
</dependency>
<dependency>
    <groupId>edu.stanford.nlp</groupId>
    <artifactId>stanford-corenlp</artifactId>
    <version>3.7.0</version>
    <classifier>models</classifier>
</dependency>
</dependencies>
```
如果你想从阿拉伯语，中文，德语或西班牙语中获得Maven的语言模型，请将其添加到您的pom.xml：
```xml
<dependency>
    <groupId>edu.stanford.nlp</groupId>
    <artifactId>stanford-corenlp</artifactId>
    <version>3.7.0</version>
    <classifier>models-chinese</classifier>
</dependency>
```
将“models-chinese”替换为“models-english”，“models-chinese-kbp”，“models-arabic”，“models-french”，“models-german”或“models-spanish”为其他语言！

由于Stanford CoreNLP的模型文件太大（中文的模型就几百M），建议到官网下载。如果使用MAVEN的话，最好使用阿里的MAVEN仓库，这样速度更快些。

## 通过命令行使用
### Quick Start
```
java -cp "*" -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos,lemma,ner,parse,dcoref -file input.txt
```
`-cp "*"`是加载当前路径下的所有文件（主要是JAR包）

该`-props`参数是可选的。默认情况下，Stanford CoreNLP将在您的类路径中搜索StanfordCoreNLP.properties，并使用分发中包含的默认值。

该`-annotators`参数实际上是可选的。如果你离开它，代码使用一个内建的属性文件，它可以使用以下注释器：标记化和句子分割，POS标记，缩小，NER，解析和关联解析（也就是我们在这些例子中使用的）。

如果要使用其它语言的话，
```
java -mx3g -cp "*" edu.stanford.nlp.pipeline.StanfordCoreNLP -props StanfordCoreNLP-chinese.properties -file chinese.txt -outputFormat text
```
- - - - -
参考：
<https://stanfordnlp.github.io/CoreNLP/index.html>