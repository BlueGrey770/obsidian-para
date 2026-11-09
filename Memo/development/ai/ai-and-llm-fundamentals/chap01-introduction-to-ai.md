---
type: memo
date: 2026-11-06
aliases:
  - "chap01-ren-shi-ai"
  - "Chap01. 认识AI"
---
# Chap01. 认识AI

本篇介绍了AI的一些核心概念，有利于你理解大模型开发的一些原理。

## 1.人工智能发展

**AI**，人工智能（**A**rtificial **I**ntelligence），使机器能够像人类一样思考、学习和解决问题的技术。



AI发展至今大概可以分为三个阶段：

![mt8hbw0g9o3yb5xy2prchsyvnof](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mt8hbw0g9o3yb5xy2prchsyvnof.png)
> 图注：这张图是AI发展阶段的划分示意图，清晰呈现了AI发展的三个核心阶段，以及各阶段的时间、特点与应用。图中按时间线划分，1950s-1980s是规则和符号AI的时代，特点是基于逻辑和规则、用符号表示知识推理，应用于医学诊断、化学结构分析；1980s-2010s是机器学习阶段，基于数据通过统计优化方法训练模型，涵盖监督、无监督学习，应用包括推荐引擎；2010s至今是深度学习阶段，模仿人脑结构功能，用多层神经元网络处理复杂任务，应用有图像识别、自然语言处理。该图与上下文提到的AI发展阶段内容相呼应，帮助直观理解AI的发展脉络。

其中，深度学习领域的自然语言处理(**N**atural **L**anguage **P**rocessing, NLP)有一个关键技术叫做**Transformer**，这是一种由多层感知机组成的神经网络模型，是现如今AI高速发展的最主要原因。



我们所熟知的**大模型**（**L**arge **L**anguage **M**odels, LLM），例如GPT、DeepSeek底层都是采用Transformer神经网络模型。



以GPT模型为例，其三个字母的缩写分别是Generative、Pre-trained、Transformer：

![g4k7boos5o0lkrxylleclfirncg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g4k7boos5o0lkrxylleclfirncg.png)
> 图注：图片展示了GPT模型的三个字母缩写所代表的关键技术。左侧蓝色圆圈内字母“G”代表生成式，其功能是根据上下文预测之后应该出现哪个文本，从而形成连续的文本输出。中间紫色圆圈内字母“P”代表预训练，通过大规模文本数据进行预训练，让大模型可以理解人类语言的语法、词性。右侧蓝色圆圈内字母“T”代表Transformer，它是深度学习的一种神经网络模型，多数AIGC模型都依赖于此。该图与上文介绍GPT模型三个字母缩写的内容相呼应，直观呈现了其技术含义。

那么问题来， Transformer神经网络有什么神奇的地方，可以实现如此强大的能力呢？



## 2.大模型原理

其实，最早Transformer是由Google在2017年提出的一种神经网络模型，一开始的作用是把它作为**机器翻译**的核心：

![twrqbpdjwoadz5x2obocj3zunlb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/twrqbpdjwoadz5x2obocj3zunlb.gif)
> 图注：图片展示了Transformer在机器翻译中的应用。画面中有一个黑色的立方体，标注为“Transformer”，其两侧各有一个蓝色箭头，分别指向左侧的“machine translation”和右侧的空白区域。该图与上下文紧密相关，上下文提到Transformer最早由Google在2017年提出，最初作为机器翻译的核心，此图直观呈现了Transformer在机器翻译这一特定任务中的作用，帮助理解其在语言处理领域的应用。

Transformer中提出的**注意力机制**使得神经网络在处理信息时可以根据上下内容调整对数据的理解，变得更加智能化。这不仅仅是说人类的文字，包括图片、音频数据都可以交给Transformer来处理。于是，越来越多的模型开始基于Transformer实现了各种神奇的功能。



例如，有的模型可以根据音频生成文本，或者根据文本生成音频：

![ubybbnut1o64h3xs83hcziq8nhf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ubybbnut1o64h3xs83hcziq8nhf.gif)
> 图注：图片展示了一个Transformer模型，上方标注“voice-to-text”，下方标注“Transformer”。该图位于介绍大模型原理的文档中，用于说明大模型的一种用法。文档提到有的模型可根据音频生成文本，此图即展示了Transformer模型在语音转文本（voice-to-text）任务中的应用，直观呈现了模型结构，与上下文对大模型用法的阐述相呼应。



还有的模型则可以根据文字生成图片，比如Dall-E、MidJourney：

![lgblbyikhopqomxaqoicj11wnde](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lgblbyikhopqomxaqoicj11wnde.gif)
> 图注：图片展示了text-to-image模型的结构。左侧输入“画一幅画，鱼香肉丝”，中间是Transformer模型，右侧输出箭头指向“text-to-image”。该图与上下文紧密相关，上下文提到大模型原理中，Dall - E、MidJourney等模型可以根据文字生成图片，此图直观呈现了text - to - image模型的工作流程，即输入文字指令，通过Transformer模型处理，输出图片内容。





不过，我们今天要聊的**大语言模型**（**L**arge **L**anguage **M**odels, 以下简称LLM）是对Transformer的另一种用法：**推理预测。**



LLM在训练Transformer时会尝试输入一些文本、音频、图片等信息，然后让Transformer推理接下来跟着的应该是什么内容。推理的结果会以概率分布的形式出现：

![sehrbbwfqojo1ix3bhecl8dunbe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sehrbbwfqojo1ix3bhecl8dunbe.gif)
> 图注：这张图展示了大模型推理预测的过程，左侧输入框带有“北京早餐有什么好吃____”的文本内容，该内容经蓝色箭头指向标注为“Transformer”的黑色模块，模块上方标注有“推理预测”，随后经蓝色箭头输出内容。该图对应文档中关于大模型推理原理的讲解，直观呈现了大模型接收前文文本、经Transformer模块进行推理预测的过程，辅助解释大模型如何依据前文持续生成关联内容的原理。

可能大家会有疑问：

> 仅仅是推测接下来的内容，怎么能让ChatGPT在对话中**生成大段的有关联的文字内容**呢？



其实LLM采用的就是笨办法，答案就是：持续生成



根据前文推测出接下来的一个词语后，把这个词语加入前文，再次交给大模型处理，推测下一个字，然后不断重复前面的过程，就可以生成大段的内容了：

![oywrbonfdopauwxiyvfcxqx1ntb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oywrbonfdopauwxiyvfcxqx1ntb.gif)
> 图注：图片展示了一个黑色的长方体，上面用白色字体写着“Transformer”。该图片位于介绍大模型原理中LLM持续生成内容方法的上下文之后，是对LLM采用笨办法生成大段关联文字内容这一原理的形象化说明，直观呈现了大模型的核心组件Transformer，帮助读者更好地理解LLM通过持续生成来生成大段内容的原理。

这就是为什么我们跟AI聊天的时候，它生成的内容总是一个字一个字的输出的原因了。



以上就是LLM的核心技术，Transformer的原理了\~



如果大家想要进一步搞清楚Transformer机制，可以参考以下两个视频：

<bookmark name="90分钟！清华博士带你一口气搞懂人工智能和神经网络_哔哩哔哩_bilibili" href="https://www.bilibili.com/video/BV1atCRYsE7x"></bookmark>

<bookmark name="www.youtube.com" href="https://www.youtube.com/watch?v=wjZofJX0v4M&amp;t=1169s"></bookmark>

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / AI 与大模型基础 / Chap01. 认识AI · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/F70SwQsh5ixlclkdBdVcgWUtn1c)
