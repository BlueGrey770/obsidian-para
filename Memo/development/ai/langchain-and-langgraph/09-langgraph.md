---
type: memo
date: 2026-11-06
aliases:
  - "09-LangGraph"
---
# 09-LangGraph

## 1 快速入门

### 1.1 Chain对比Graph

LangChain&LangGraph开发生态是一个模块化的语言模型应用开发框架体系，其核心架构与组件可系统化分解如下:

- 开发：使用LangChain的开源组件和第三方集成构建您的应用程序。使用LangGraph构建具有完善工作流和人工介入支持的有状态智能体
- 生产化：使用LangSmith检查、监控和评估您的应用程序，以便您可以放心地持续优化和部署
- 部署：使用LangGraph Platform将您的LangGraph应用程序转换为生产就绪的API和助手

![unpabwka3otpouxqcbzcneyhngb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/unpabwka3otpouxqcbzcneyhngb.png)
> 图注：这张图展示了LangGraph相关工具的架构与分类结构，左侧分为Deployment、Components、Architecture三个类别，其中Deployment下的LangGraph Platform为商业产品，Components下的Integrations为开源产品，Architecture类别下包含LangChain与LangGraph两款开源产品；右侧的LangSmith为商业产品，其功能涵盖Debugging、Playground、Prompt Management、Annotation、Testing、Monitoring等，该图清晰呈现了不同Lang系列工具的定位、所属类别及商业/开源属性。



**LangChain**是一个通用的智能体编排工具，它把工作编排成一系列的工作节点，用一条工作链把这些节点连接起来，组织成一张有向无环图(DAG)

它的工作方式适合比较简单的智能体（比如：提示词-->调用模型-->结果优化），一旦涉及到一些复杂的流程，就有些力不从心了。比如下面的场景：

- **动态调整流程**‌：根据中间结果决定下面的分支路径
- **分支与并行**：一些步骤可以拆分成不同的分支，并行完成后整合结果
- **多人协作**‌：不同步骤可以用不同模型/API（比如GPT写文案+Stable Diffusion画图）
- **人工介入**：在某些非常关键的位置，必须由人类决定的点，可以人工介入，然后继续走流程

![byuabi761ohdkvxuk6lc1ynznhd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/byuabi761ohdkvxuk6lc1ynznhd.png)
> 图注：图片展示了一个基于Graph结构的流程图，用于说明动态调整流程、分支与并行、多人协作及人工介入等场景。流程从“__start__”开始，分为两条路径，一条是“磨咖啡豆”后“冲咖啡”，再询问是否加糖，若加糖则“加糖”后结束，若不加糖则结束；另一条是“heat water”（加热水），若水烧开了则“get boil water”，若水没开则循环返回“heat water”。该图直观呈现了LangGraph框架下流程的动态变化与分支处理，与上下文介绍的LangGraph核心创新相契合。

**LangGraph**是由LangChain团队开发的开源框架，专为构建基于大型语言模型(LLM)的有状态、多参与者应用程序而设计

其核心创新在于采用图结构(Graph)作为计算模型，突破了传统有向无环图(DAG)的限制，支持循环工作流和动态状态管理

用**节点（Node）**表示任务步骤，用**边（Edge）**连接节点，并通过**状态（State）**在不同节点之间传递信息

![kykwbm1dcox2d7xnbvoctlyknzb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kykwbm1dcox2d7xnbvoctlyknzb.png)
> 图注：这张图展示了LangGraph的基础结构，左侧蓝色矩形的“Start”作为流程起点，通过连线依次连接黄色的“Node 1”和“Node 2”两个任务步骤节点，最右侧为蓝色的“Stop”流程终点。此外，图中存在一条从两个节点指向下方绿色“State”模块的箭头，代表节点在执行过程中会生成并传递状态信息，该图对应上下文里关于LangGraph采用图结构、以节点表示任务步骤且通过状态传递信息的核心内容。

LangGraph主要可以应用在下面场景中：

- **多智能体协作：**构建需要多个智能体协同工作的系统，如自动化客服、智能推荐等
- **复杂工作流管理：**实现包含循环、条件分支和并发执行的复杂任务流程，如文档处理、数据分析等
- **人机协作系统：**设计需要人工审核或干预的应用，如内容审核、决策支持等
- **长期任务管理：**处理需要长时间运行并保持状态的任务，如自动化测试、持续集成等



### 1.2 图的基本组成

在LangGraph中主要是搞懂**图(Graph)**的概念，而图主要由三部分组成：

- **State（状态）：在节点之间传递数据的对象**，用于存储所有上下文信息，比如输入参数、输出参数等等
- **Node（节点）：**表示流程中的某一个步骤，主要体现为函数，它的输入和输出一般都是状态对象
- **Edge（边）：**定义节点之间的连接关系和执行顺序

![m2pqbt9vpo3urcxtsgqcgjg4nqo](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/m2pqbt9vpo3urcxtsgqcgjg4nqo.png)
> 图注：图片展示了基于LangGraph实现的简单两位数计算图。图中“__start__”为起始节点，有两条分支，分别指向“得到热水子图”和“磨咖啡豆”节点。这两个节点的输出再共同指向“冲咖啡”节点。之后，“冲咖啡”节点的输出指向“加糖子图1”节点，最后“加糖子图1”的输出到达“__end__”节点。该图直观呈现了图中节点的连接关系和执行顺序，与上下文介绍的图由状态、节点、边组成，以及后续的代码编写步骤相呼应。

![bf33brl48ojivcxkjcgcigvcnbg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bf33brl48ojivcxkjcgcigvcnbg.png)
> 图注：图片展示了基于LangGraph实现的两位数计算图示。图中包含“__start__”起始节点，“询问是否加糖1”节点，以及“加糖”和“__end__”节点。其中，“询问是否加糖1”节点有“是”和“否”两个分支，分别指向“加糖”和“__end__”节点。


### 1.3 入门案例

#### 1.3.1 案例需求

基于LangGraph实现一个简单的两位数计算的图，图示如下：

```Python
开始：输入x=1，y=2
执行加法节点：x=x + y   x=3， y=2
执行乘法节点：x=x * y   x=6， y=2
结束：打印x = 6
```

![ieqwbcwkfodmfrxlbt1cukatnng](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ieqwbcwkfodmfrxlbt1cukatnng.png)
> 图注：这张图是基于LangGraph实现两位数计算的入门案例流程，对应案例需求中的计算图。图中从上至下依次设置了__start__、add_node、multi_node、__end__的流程节点，节点间通过箭头连接明确执行顺序。对应流程的具体操作为：从起始节点__start__输入两位数x与y，经add_node执行加法运算x=x+y并输出结果，再经multi_node执行乘法运算x=x*y并输出结果，最终由__end__节点结束流程。

#### 1.3.2 准备环境

基于conda创建新的虚拟环境

```Shell
conda create --name py_langgraph python=3.13.5
```

创建工程，安装LangGraph

```Shell
pip install -U langgraph==1.0.4
pip install -U langchain==1.1.0
pip install -U langchain-openai==1.1.0
pip install langchain-ollama
pip install ChatOllama
pip install matplotlib
```

#### 1.3.3 编写代码

```Python
# 构建第一张图(状态、节点、边)
import os
from typing import TypedDict

from langgraph.constants import START, END
from langgraph.graph import StateGraph
from langgraph.graph.state import CompiledStateGraph

from mk_graph import mk_graph


# 1) 定义状态: 一个类 需要在里面定义属性 需要继承TypedDict
class CalState(TypedDict):  # {"x":1 , "y":2}
    x: int
    y: int


# 2) 定义节点: 一个函数 里面需要定义任务的执行流程 参数和返回值都是State对象
def add_node(calState: CalState) -> CalState:
    calState["x"] = calState["x"] + calState["y"]
    return calState


def multi_node(calState: CalState) -> CalState:
    calState["x"] = calState["x"] * calState["y"]
    return calState


# 3) 画图
def build_graph() -> CompiledStateGraph:
    # 3-1 创建图的对象
    graph_builder = StateGraph(CalState)

    # 3-2 声明节点对象到图中  参数1: 节点名字  参数2: 节点对象
    graph_builder.add_node("add_node", add_node)
    graph_builder.add_node("multi_node", multi_node)

    # 3-3) 添加边到图中, 必须指定边的开始和结尾
    graph_builder.add_edge(START, "add_node")
    graph_builder.add_edge("add_node", "multi_node")
    graph_builder.add_edge("multi_node", END)

    # 3-4) 编译图
    graph = graph_builder.compile()
    return graph


if __name__ == '__main__':
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 执行图
    calState = {"x": 1, "y": 2}
    calState = graph.invoke(calState)
    print(calState["x"], calState["y"])
```

执行一个LangGraph项目要完成以下基本步骤：

1. 定义状态类，需要继承TypedDict类
2. 定义节点函数，参数都是状态类
3. 定义建图函数，在函数中创建图，并拼接节点和边
4. 编译图
5. 执行图

## 2  核心组件

LangGraph的核心组件有：图（Gragh）、**状态（State）、节点（Node）、边（Edge）**、记忆（Memory），下面依次进行讲解

### 2.1 图

图是节点和边的集合，LangGraph里的图有两个主要实现：

- StateGraph：它是LangGraph的核心结构，用于需要维护长期状态的场景，比较常用
- MessageGraph：它是StateGraph的子类，一般用于多轮对话的场景，使用的相对较少

StateGraph默认有一个START节点和一个END节点，编写一个**最简化**的图

![ae4sbw02voybjxxw9y5cnxf6ncc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ae4sbw02voybjxxw9y5cnxf6ncc.png)
> 图注：图片展示了LangGraph中StateGraph的最简化图结构。图中有两个节点，上方节点为“__start__”，下方节点为“end”，二者之间有一条虚线箭头连接。该图与上下文紧密相关，上下文提到StateGraph默认有一个START节点和一个END节点，此图直观呈现了这一结构，帮助理解StateGraph在最简化场景下的基本组成。

代码如下：

```Python
import os

from langgraph.constants import START, END
from langgraph.graph import StateGraph
from langgraph.graph.state import CompiledStateGraph

from mk_graph import mk_graph


# 建图: 自定义一个函数, 实现图中各部分的组装
def build_graph() -> CompiledStateGraph:
    # 1) 创建图
    graph_builder = StateGraph(dict)

    # 3) 添加边(一条边必须以节点开始和结束 START和END是两个特殊的节点)
    graph_builder.add_edge(START, END)

    # 4) 编译图
    graph = graph_builder.compile()
    return graph


if __name__ == '__main__':
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])
```

### 2.2 状态

状态用于在复杂任务或对话系统中存储和传递上下文信息，它的数据类型一般是dict或者是继承TypedDict(规范类型，方便数据检查)

```Python
# 构建第一张图(状态、节点、边)
import os
from typing import TypedDict

from langgraph.constants import START, END
from langgraph.graph import StateGraph
from langgraph.graph.state import CompiledStateGraph
from mk_graph import mk_graph


# 定义状态: 自定义一个类继承TypedDict, 然后声明属性和类型即可
class CalState(TypedDict):
    x: int
    y: int


# 定义节点：自定义一个函数，实现节点逻辑
def add_node(calState: CalState) -> CalState:
    calState['x'] = calState['x'] + calState['y']
    return calState


def multi_node(calState: CalState) -> CalState:
    calState['x'] = calState['x'] * calState['y']
    return calState


# 建图: 自定义一个函数, 实现图中各部分的组装
def build_graph() -> CompiledStateGraph:
    # 1) 创建图
    graph_builder = StateGraph(CalState)

    # 2) 添加节点(参数1: 节点名称  参数2: 节点函数名)
    graph_builder.add_node("add_node", add_node)
    graph_builder.add_node("multi_node", multi_node)

    # 3) 添加边(一条边必须以节点开始和结束 START和END是两个特殊的节点)
    graph_builder.add_edge(START, "add_node")
    graph_builder.add_edge("add_node", "multi_node")
    graph_builder.add_edge("add_node", END)

    # 4) 编译图
    graph = graph_builder.compile()
    return graph


if __name__ == '__main__':
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    calState = {"x": 1, "y": 2}
    calState = graph.invoke(calState)
    print(calState["x"], calState["y"])
```

### 2.3 节点

节点代表一个具体的操作单元，一般封装成一个函数执行业务逻辑，它的输入和输出都是State，一般来说，有以下八种节点：

<sheet sheet-id="qPEdTb" token="MKH2sh0oeh6FXRtIzN7cxmcOng5"></sheet>

这里只展示基本节点，流程控制类的节点放到下个小节和对应的边一起介绍

**① 函数节点：**本质上就是一个通用的python函数

```Python
# 定义一个函数作为节点
def my_node(state)->state:
    """ 简单计算函数  使用state进行参数传递"""
    state["z"] = state["x"] + state["y"]
    return state
```

**② 工具节点:** 封装外部工具或API

```Python
def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"
```

**③大模型节点：**调用大模型生成结果，如果需要使用工具，可以调用时加以说明

```Python
def configLLM(state):
    agent = create_react_agent(
        model=llm, # 模型
        tools=[get_weather], # 工具
        prompt="You are a helpful assistant" # 系统提示词
    )
    #调用大模型
    result = agent.invoke(
        {"messages": [{"role": "user", "content": state["query"]}]}  # 用户提示词
    )
    state["answer"] = result["messages"][-1].content
    return state
```

### 2.4 边

边是连接节点的逻辑通道，用来控制工作流的执行顺序和数据流向，主要有以下两种类型

- 普通边：直接连接两个节点，无条件跳转
- 条件边：基于状态动态选择路径，通过函数返回目标节点名

![s9kubtfxsogxmkxd5f6cuyx0nme](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/s9kubtfxsogxmkxd5f6cuyx0nme.png)
> 图注：这张图是展示LangGraph核心组件里边的类型的流程图，内容结合了文档中边的相关说明，清晰呈现了边的两种类型：普通边和条件边。图中__start__节点通过普通边指向heatUp节点；heatUp节点通过条件边，当判断为“yes”时，通过带有“普通边”标识的实线箭头指向getWater节点，当判断为“no”时，通过虚线箭头回到heatUp节点；getWater节点再通过普通边指向__end__节点，直观体现了普通边的无条件跳转和条件边的动态路径选择的特点。

```Python
import os
from typing import TypedDict

from langgraph.constants import START, END
from langgraph.graph import StateGraph
from langgraph.graph.state import CompiledStateGraph
from mk_graph import mk_graph


# 定义状态
class WaterState(TypedDict):
    temperature: int


# 定义节点: 加热
def 加热(waterState: WaterState) -> WaterState:
    print(f"当前水温:{waterState['temperature']},开始加热")
    waterState['temperature'] = waterState['temperature'] + 10
    print(f"加热完毕,当前水温:{waterState['temperature']}")
    return waterState


def 取水(waterState: WaterState) -> WaterState:
    print(f"水可以用了,温度{waterState['temperature']}")
    return waterState


# 定义条件边
def 水温判断(waterState: WaterState):
    if waterState['temperature'] == 100:
        print(f"判断水温为:{waterState['temperature']},返回yes")
        return "yes"
    else:
        print(f"判断水温为:{waterState['temperature']},返回no")
        return "no"


# 建图: 自定义一个函数, 实现图中各部分的组装
def build_graph() -> CompiledStateGraph:
    # 1) 创建图
    graph_builder = StateGraph(WaterState)

    # 2) 添加节点
    graph_builder.add_node("加热", 加热)
    graph_builder.add_node("取水", 取水)

    # 3) 添加边
    # 普通边 add_edge(开始节点名称, 结束节点名称)
    graph_builder.add_edge(START, "加热")

    # 条件边 add_conditional_edges(开始节点名称, 条件边, 路由表)
    graph_builder.add_conditional_edges(
        "加热",  # 开始节点名称
        水温判断,  # 条件边, 也是一个函数,会返回下一步的理由表示
        {"yes": "取水", "no": "加热"}  # 路由表-- {路由标识:节点名称}
    )

    # 普通边 add_edge(开始节点名称, 结束节点名称)
    graph_builder.add_edge("取水", END)

    # 4) 编译图
    graph = graph_builder.compile()
    return graph


if __name__ == '__main__':
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    waterState = {"temperature": 40}
    calState = graph.invoke(waterState)
```

### 2.5 记忆

`LangGraph`允许带有记忆功能，是通过`LangGraph.checkpoint.memory`来实现的

```Python
from typing import Annotated
from langchain_ollama import ChatOllama
from langgraph.checkpoint import memory
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from typing_extensions import TypedDict


# 定义状态类
class State(TypedDict):
    # 1. 基础类型：list - 表示该字段的值必须是一个列表
    #    存储对话消息，如[{"role": "user", "content": "..."}]
    # 2. 附加元数据：add_messages - 这是langgraph的状态更新器，作用是：
    #    当向messages字段写入新消息时，不会直接覆盖原有列表，而是将新消息追加到列表末尾
    messages: Annotated[list, add_messages]


# 初始化Ollama聊天模型
llm = ChatOllama(model="qwen2.5:7b")


# 定义聊天机器人节点函数
def chatbot(state: State):
    # 调用LLM处理当前消息列表，并将响应添加到消息中
    return {"messages": [llm.invoke(state["messages"])]}


# 构建图
def buildGraph():
    # 创建状态图实例
    graphBuilder = StateGraph(State)

    # 实例化内存保存器，用于保存对话历史
    memory = MemorySaver()

    # 添加聊天机器人节点
    graphBuilder.add_node("chatbot", chatbot)
    graphBuilder.add_edge(START, "chatbot")
    graphBuilder.add_edge("chatbot", END)

    # 编译图时指定内存检查点，使图能够保存和恢复状态
    #graph = graphBuilder.compile()
    graph = graphBuilder.compile(checkpointer=memory)
    return graph


# 主程序入口
if __name__ == "__main__":
    graph = buildGraph()
    config = {"configurable": {"thread_id": "1"}}  # 配置线程ID，用于区分不同对话会话
    while True:
        # 获取用户输入
        userInput = input("User: ")
        # 构建初始状态，包含用户消息
        state = {"messages": [{"role": "user", "content": userInput}]}
        # 调用图处理当前状态，使用配置来保持对话上下文
        result = graph.invoke(state, config=config)
        print(f"result========={result}")
        # 打印助手的回复
        print("Assistant:", result["messages"][-1].content)
```

## 3 流程控制

LangGraph的流程控制方式：分支、循环、并行、人工介入、子图

比如说我们要冲一杯咖啡，有下面几个步骤必须要考虑：

- **磨咖啡豆**
- **烧热水**（这里有分支和循环，如果水温不够，要继续加热，一直到水烧开为止）
- **冲咖啡**（把前两步结果合并）
- **决定是否加糖**（根据用户反馈调整输出）

这个过程可以用下图表示：

![vw9cbjrc7oietgxwboocrnghnje](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vw9cbjrc7oietgxwboocrnghnje.png)
> 图注：图片展示了冲咖啡流程的图示。从“__start__”开始，分为“磨咖啡豆”和“烧水”两条路径。烧水路径中，若水烧开则进入“冲咖啡”，否则继续加热；磨咖啡豆路径直接进入“冲咖啡”。冲咖啡后询问是否加糖，若加糖则为“加糖咖啡”，不加糖则为“无糖咖啡”，最后都到达“__end__”。该图与上下文介绍的冲咖啡流程控制方式相呼应，直观呈现了流程中分支、循环等控制逻辑。

### 3.1 分支

通过综合运用条件边和条件节点，可以控制graph的分支，我们把关注点放在案例的**烧开水部分**

![tiwjb38pmo4kisx6wukcu3ypn9e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tiwjb38pmo4kisx6wukcu3ypn9e.png)
> 图注：这张图是LangGraph中描述烧开水流程的分支示意图，对应文档里冲咖啡案例的分支控制内容。流程从“__start__”节点开始，进入“烧水”环节；烧水后根据结果分为两个分支，若水烧开则流向“冲咖啡”节点，最后汇至“__end__”节点，若水没开则跳转至“需要继续加热”节点，加热后重新汇入终点，这一设计结合了分支与循环逻辑，能够满足烧开水时水温不足需反复加热、达到要求后进入后续步骤的实际需求。

根据上述流程实现代码：

```Python
import os

from langgraph.graph import START, StateGraph, END
from typing_extensions import TypedDict

from mk_graph import mk_graph


# 添加状态类
class State(TypedDict):
    水温: int
    产物: str


# 烧水节点
def 烧水节点(state):
    print(f"烧水节点==当前水温:{str(state)},开始烧水")
    state["水温"] = state["水温"] + 10
    if state["水温"] >= 100:
        state["水温"] = 100
        state["产物"] = "烧开的水"
    print(f"烧水节点==烧水之后,温度:{str(state)}")
    return state


# 按照温度处理水的边
def 水温判断(state):
    if state["水温"] == 100:
        print(f"水温判断条件==当前水温{state['水温']},水烧开了")
        return "水烧开了"
    else:
        print(f"水温判断条件==当前水温{state['水温']},水没开")
        return "水没开"


# 冲咖啡节点
def 冲咖啡节点(state):
    print("冲咖啡节点==冲咖啡之前:" + str(state))
    state["产物"] = "咖啡"
    print("冲咖啡节点==冲咖啡之后:" + str(state))
    return state


# 需要继续加热函数
def 继续加热节点(state):
    state["产物"] = "没烧开的水"
    print("继续加热节点==继续加热" + str(state))
    return state


# 构建图
def build_graph():
    # 初始化图
    graphBuilder = StateGraph(State)

    # 添加节点
    graphBuilder.add_node("烧水节点", 烧水节点)
    graphBuilder.add_node("冲咖啡节点", 冲咖啡节点)
    graphBuilder.add_node("继续加热节点", 继续加热节点)

    # 添加边
    graphBuilder.add_edge(START, "烧水节点")
    graphBuilder.add_conditional_edges("烧水节点", 水温判断, {"水烧开了": "冲咖啡节点", "水没开": "继续加热节点"})
    graphBuilder.add_edge("冲咖啡节点", END)
    graphBuilder.add_edge("继续加热节点", END)

    # 编译图
    graph = graphBuilder.compile()
    return graph


if __name__ == "__main__":
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    state = {"水温": 28, "产物": "没烧开的水"}
    result = graph.invoke(state)
    print(result)
```

① 如果初始水温设置到30度，运行结果如下：

```Python
烧水节点==当前水温:{'水温': 28, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 38, '产物': '没烧开的水'}
水温判断条件==当前水温38,水没开
继续加热节点==继续加热{'水温': 38, '产物': '没烧开的水'}
{'水温': 38, '产物': '没烧开的水'}
```

② 如果将水温设置到90度，经过了加温会到达100度，也就是满足了冲咖啡的条件

```Python
烧水节点==当前水温:{'水温': 90, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 100, '产物': '烧开的水'}
水温判断条件==当前水温100,水烧开了
冲咖啡节点==冲咖啡之前:{'水温': 100, '产物': '烧开的水'}
冲咖啡节点==冲咖啡之后:{'水温': 100, '产物': '咖啡'}
{'水温': 100, '产物': '咖啡'}
```

### 3.2 循环

通过合理设置条件节点的跳转，可以构造循环。

烧开水每次增加水温10度，可能一次结束，更可能需要多次，这时就需要循环，我们继续把关注点集中在烧开水部分

**实际上， LangGraph 中的“循环”本质上不是 while，而是「条件分支 + 指向自身的边」**  

![rlc1bp65uoavbpxmbxic6n3ynue](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rlc1bp65uoavbpxmbxic6n3ynue.png)
> 图注：这是LangGraph流程控制中关于烧开水的循环逻辑示意图，属于冲咖啡流程里的烧开水环节。图中以“start”为起始，指向核心节点“烧水”；当判断“水烧开了”时，流程进入“end”结束节点；若判断“水没开”，则通过指向自身的虚线箭头，让流程回到“烧水”节点，实现循环加温，这对应了文档中所说的“条件分支+指向自身的边”的循环本质，直观展现了水温不足时反复加温、达标后结束流程的控制逻辑。

代码实现如下

```Python
import os

from langgraph.graph import START, StateGraph, END
from typing_extensions import TypedDict

from mk_graph import mk_graph


# 添加状态类
class State(TypedDict):
    水温: int
    产物: str


# 烧水节点
def 烧水节点(state):
    print(f"烧水节点==当前水温:{str(state)},开始烧水")
    state["水温"] = state["水温"] + 10
    if state["水温"] >= 100:
        state["水温"] = 100
        state["产物"] = "烧开的水"
    print(f"烧水节点==烧水之后,温度:{str(state)}")
    return state


# 按照温度处理水的边
def 水温判断(state):
    if state["水温"] == 100:
        print(f"水温判断条件==当前水温{state['水温']},水烧开了")
        return "水烧开了"
    else:
        print(f"水温判断条件==当前水温{state['水温']},水没开")
        return "水没开"


# 冲咖啡节点
def 冲咖啡节点(state):
    print("冲咖啡节点==冲咖啡之前:" + str(state))
    state["产物"] = "咖啡"
    print("冲咖啡节点==冲咖啡之后:" + str(state))
    return state

# 构建图
def build_graph():
    # 初始化图
    graphBuilder = StateGraph(State)

    # 添加节点
    graphBuilder.add_node("烧水节点", 烧水节点)
    graphBuilder.add_node("冲咖啡节点", 冲咖啡节点)

    # 添加边
    graphBuilder.add_edge(START, "烧水节点")
    graphBuilder.add_conditional_edges("烧水节点", 水温判断, {"水烧开了": "冲咖啡节点", "水没开": "烧水节点"})
    graphBuilder.add_edge("冲咖啡节点", END)

    # 编译图
    graph = graphBuilder.compile()
    return graph

if __name__ == "__main__":
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    state = {"水温": 40, "产物": "没烧开的水"}
    result = graph.invoke(state)
    print(result)
```

运行结果如下

```Python
烧水节点==当前水温:{'水温': 40, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 50, '产物': '没烧开的水'}
水温判断条件==当前水温50,水没开
烧水节点==当前水温:{'水温': 50, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 60, '产物': '没烧开的水'}
水温判断条件==当前水温60,水没开
烧水节点==当前水温:{'水温': 60, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 70, '产物': '没烧开的水'}
水温判断条件==当前水温70,水没开
烧水节点==当前水温:{'水温': 70, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 80, '产物': '没烧开的水'}
水温判断条件==当前水温80,水没开
烧水节点==当前水温:{'水温': 80, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 90, '产物': '没烧开的水'}
水温判断条件==当前水温90,水没开
烧水节点==当前水温:{'水温': 90, '产物': '没烧开的水'},开始烧水
烧水节点==烧水之后,温度:{'水温': 100, '产物': '烧开的水'}
水温判断条件==当前水温100,水烧开了
冲咖啡节点==冲咖啡之前:{'水温': 100, '产物': '烧开的水'}
冲咖啡节点==冲咖啡之后:{'水温': 100, '产物': '咖啡'}
{'水温': 100, '产物': '咖啡'}
```

### 3.3 并行

在一个graph中，某些节点可以并行，**例如：**在煮咖啡任务中，烧水和磨咖啡豆两个任务，互不干扰，可以同时进行

这一小节，我们把关注点转移到烧开水和磨咖啡豆两个任务的并行执行和结果合并上

![uzuqbiiddorsybxegu7cqdrjnwd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/uzuqbiiddorsybxegu7cqdrjnwd.png)
> 图注：这张图是煮咖啡任务的流程控制示意图，对应文档中3.3并行小节的内容。图中以`__start__`作为起始节点，分支出两条并行路径，分别指向“烧水节点”和“磨咖啡豆节点”；两个并行节点完成后，箭头共同汇入“冲咖啡节点”，最终由“冲咖啡节点”指向`__end__`结束节点。该图直观展示了烧水与磨咖啡豆这两个并行任务的执行逻辑，与文档中关于并行节点的说明内容相呼应，体现了并行节点执行后再合并的流程设计。

代码实现

```Python
import os
from langgraph.graph import START, StateGraph, END
from typing_extensions import TypedDict, Annotated

from mk_graph import mk_graph


# 自定义规则
def rule(current: str, update: str) -> str:
    return update if current == 0 else current


# 添加状态类
class State(TypedDict):
    水温: int
    咖啡: str
    分数: Annotated[int, rule] # 更新冲突的时候,走自定义规则


# 烧水节点
def 烧水节点(state):
    # print(f"烧水节点==当前水温:{str(state)},开始烧水")
    # state["水温"] = 100
    # print(f"烧水节点==烧水之后,温度:{str(state)}")
    return {"水温": 100, "分数": 50}


# 磨咖啡豆节点
def 磨咖啡豆节点(state):
    # print("磨咖啡豆节点==之前:" + str(state))
    # # state["咖啡"] = "咖啡粉"
    # print("磨咖啡豆节点==之后:" + str(state))
    return {"咖啡": "咖啡粉", "分数": 60}


# 冲咖啡节点
def 冲咖啡节点(state):
    # print("冲咖啡节点==之前:" + str(state))
    # state["咖啡"] = "完美咖啡"
    # print("冲咖啡节点==之后:" + str(state))
    return {"咖啡": "完美咖啡"}


# 构建图
def build_graph():
    # 初始化图
    graphBuilder = StateGraph(State)

    # 添加节点
    graphBuilder.add_node("烧水节点", 烧水节点)
    graphBuilder.add_node("磨咖啡豆节点", 磨咖啡豆节点)
    graphBuilder.add_node("冲咖啡节点", 冲咖啡节点)

    # 添加边
    graphBuilder.add_edge(START, "烧水节点")
    graphBuilder.add_edge(START, "磨咖啡豆节点")
    graphBuilder.add_edge(["烧水节点", "磨咖啡豆节点"], "冲咖啡节点")
    graphBuilder.add_edge("冲咖啡节点", END)

    # 编译图
    graph = graphBuilder.compile()
    return graph


if __name__ == "__main__":
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    state = {"水温": 50, "咖啡": "咖啡豆"}
    result = graph.invoke(state)
    print(result)
```

注意下面的细节：

1. 并行节点需要使用列表表示  graphBuilder.add_edge(["烧水节点", "磨咖啡豆节点"], "冲咖啡节点")
2. 并行节点往State写不同key，LangGraph可以自动合并状态，无需特殊的处理
3. 并行节点往State写相同key，LangGraph会报错，此时需要针对冲突字段，需要自定义更新规则 Annotated[int, rule]

### 3.4 人工介入

有些关键环节需要人工决策，这是需要引入人工介入的方式

这一小节，我们把关注点转移到案例的人工决定是否加糖的步骤上

人工决策的关键点有三个：

- 必须引入一个带`interrupt`功能的节点
- 必须人工决策后**再次**`invoke`
- `graph`必须带有`memory`

![dc1nbfbq5of8lgxldktcjh7hnhd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dc1nbfbq5of8lgxldktcjh7hnhd.png)
> 图注：图片展示了煮咖啡任务的流程图。从“__start__”开始，依次经过“冲咖啡”节点，然后到达“询问是否加糖”节点，该节点有“是”和“否”两个分支，若选择“是”则进入“加糖”节点，最后到达“__end__”节点；若选择“否”则直接到达“__end__”节点。此图与上下文紧密相关，直观呈现了案例中人工决定是否加糖这一关键环节的流程，是人工介入方式的示例。

代码实现如下:

```Python
import os

from langgraph.graph import START, StateGraph, END
from typing_extensions import TypedDict
from langgraph.types import Command, interrupt
from langgraph.checkpoint.memory import InMemorySaver

from mk_graph import mk_graph


# 添加状态类
class State(TypedDict):
    产物: str
    是否加糖: str


# 冲咖啡函数
def 冲咖啡(state):
    print("开始冲咖啡")
    return {"产物": "咖啡"}


# 中断节点，里面必须包含一个interrupt
def 询问是否加糖(state):
    human_response = interrupt("")
    return {"是否加糖": human_response}


# 根据人类是否加糖的反馈，选择不同分支
def 是否加糖分支(state):
    if state["是否加糖"] == "是":
        return "是"
    elif state["是否加糖"] == "否":
        return "否"

# 加糖函数
def 加糖(state):
    return {"产物": "加糖咖啡"}


def build_graph():
    # 初始化图
    graph_builder = StateGraph(State)
    # 初始化记忆。使用人工介入必须带有记忆，否则图中断执行后无法正常继续
    checkpointer = InMemorySaver()
    # 添加节点
    graph_builder.add_node("冲咖啡", 冲咖啡)
    graph_builder.add_node("询问是否加糖", 询问是否加糖)
    graph_builder.add_node("加糖", 加糖)
    # 添加边
    graph_builder.add_edge(START, "冲咖啡")
    graph_builder.add_edge("冲咖啡", "询问是否加糖")
    graph_builder.add_conditional_edges("询问是否加糖", 是否加糖分支, {"是": "加糖", "否": END})
    graph_builder.add_edge("加糖", END)
    # 编译图
    graph = graph_builder.compile(checkpointer=checkpointer)
    return graph


if __name__ == "__main__":
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    state: State = {"产物": "开水", "是否加糖": ""}
    config = {"configurable": {"thread_id": "1"}}

    # 启动工作流
    state = graph.invoke(state, config)
    
    # 中断后，接收用户输入
    userDecision = input("先生您好，请问您的咖啡需要加糖吗？(是/否):")
    
    # 使用Command函数向graph提供用户输入的数据，继续工作流
    result = graph.invoke(Command(resume=userDecision), config=config)
    print(result)
```

### 3.5 子图

随着项目演进，功能不断增多，用单个`graph`已经太过复杂，而且也不便于调试。

有必要把部分内聚性强的功能块独立出来，做成子图

- 烧开水子图
- 询问加糖子图
- 总图

![rhkmbva41ofmilxxzmnccvs9nmf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rhkmbva41ofmilxxzmnccvs9nmf.png)
> 图注：这张煮咖啡任务的LangGraph流程图，以__start__为起始节点，分为并行子图与后续处理流程。并行子图由烧水相关节点构成，包含heat water节点，其分支标注“水烧开了”指向get boil water节点，若“水没开”则回流至heat water节点；磨咖啡豆节点与并行子图的get boil water节点最终都指向冲咖啡节点。冲咖啡节点后接询问是否加糖节点，该节点的“是”分支指向加糖节点，“否”分支直接指向__end__结束节点，加糖节点也最终指向__end__。此图对应文档中并行、人工介入等内容，清晰展现了煮咖啡任务的节点流转逻辑。

现在整个案例已经有了完整的处理流程，整体代码如下：

```Python
import os

from langgraph.graph import START, StateGraph, END
from typing_extensions import TypedDict
from typing import Annotated
from langgraph.types import Command, interrupt
from langgraph.checkpoint.memory import InMemorySaver

from mk_graph import mk_graph


# from tools import showGraph

# 合并温度值的方式，取其大者
def updateTempreture(left, right):
    return max(left, right)


# 合并产物的方式，优先取工序靠后的产物
def updateProduct(left, right):
    if left == "加糖咖啡" or right == "加糖咖啡":
        return "加糖咖啡"
    elif left == "咖啡" or right == "咖啡":
        return "咖啡"
    elif left == "开水" or right == "开水":
        return "开水"
    elif left == "温水" or right == "温水":
        return "温水"
    else:
        return "凉水"


# 合并咖啡固体的方式，优先取工序靠后的产物
def updateSolid(left, right):
    if left == "咖啡粉" or right == "咖啡粉":
        return "咖啡粉"
    else:
        return "咖啡豆"


def updateSugur(left, right):
    if left == "是" or right == "是":
        return "是"
    else:
        return "否"


# 添加状态类
class State(TypedDict):
    水温: Annotated[int, updateTempreture]
    产物: Annotated[str, updateProduct]
    咖啡固体: Annotated[str, updateSolid]
    是否加糖: Annotated[str, updateSugur]


# 磨咖啡豆函数
def 磨咖啡豆(state):
    print("*" * 80)
    print("磨咖啡豆之前:" + str(state))
    state["咖啡固体"] = "咖啡粉"
    print("磨咖啡豆之后:" + str(state))
    return state


# 烧水函数
def 烧水(state):
    print("*" * 80)
    print("烧水之前:" + str(state))
    state["产物"] = "没烧开的水"
    state["水温"] = state["水温"] + 10
    if state["水温"] > 100:
        state["水温"] = 100
        state["产物"] = "开水"
    print("烧水之后:" + str(state))
    return state


# 按照温度处理水的条件函数
def 按温度处理水(state):
    if state["水温"] == 100:
        return "水烧开了"
    else:
        return "水没开"


# 得到开水函数，注意这个函数没有实际作用，只是便于展示整个流程
def 得到开水(state):
    return state


# 冲咖啡函数
def 冲咖啡(state):
    print("冲咖啡之前:" + str(state))
    state["产物"] = "咖啡"
    print("冲咖啡之后:" + str(state))
    return state


# 中断节点，里面必须包含一个interrupt
def 询问是否加糖(state):
    human_response = interrupt("")
    state["是否加糖"] = human_response
    return state


# 根据人类是否加糖的反馈，选择不同分支
def 是否加糖分支(state):
    # print(state)
    if state["是否加糖"] == "是":
        return "是"
    elif state["是否加糖"] == "否":
        return "否"


# 加糖函数
def 加糖(state):
    print("加糖之前:" + str(state))
    state["产物"] = "加糖咖啡"
    print("加糖之后:" + str(state))
    return state


def build_graph():
    # 初始化图
    checkpointer = InMemorySaver()

    # 烧水子图
    graphBuilder1 = StateGraph(State)
    graphBuilder1.add_node("heat water", 烧水)
    graphBuilder1.add_node("get boil water", 得到开水)
    
    graphBuilder1.add_edge(START, "heat water")
    graphBuilder1.add_conditional_edges("heat water", 按温度处理水,
                                        {"水烧开了": "get boil water", "水没开": "heat water"})
    graph1 = graphBuilder1.compile(checkpointer=checkpointer)

    # 加糖子图
    graphBuilder2 = StateGraph(State)
    graphBuilder2.add_node("询问是否加糖", 询问是否加糖)
    graphBuilder2.add_node("加糖", 加糖)

    graphBuilder2.add_edge(START, "询问是否加糖")
    graphBuilder2.add_conditional_edges("询问是否加糖", 是否加糖分支, {"是": "加糖", "否": END})
    graphBuilder2.add_edge("加糖", END)
    graph2 = graphBuilder2.compile(checkpointer=checkpointer)

    # 总图
    graphBuilder = StateGraph(State)
    graphBuilder.add_node("磨咖啡豆", 磨咖啡豆)
    graphBuilder.add_node("冲咖啡", 冲咖啡)
    graphBuilder.add_node("烧水子图", graph1)
    graphBuilder.add_node("加糖子图", graph2)

    graphBuilder.add_edge(START, "磨咖啡豆")
    graphBuilder.add_edge(START, "烧水子图")

    graphBuilder.add_edge(["烧水子图", "磨咖啡豆"], "冲咖啡")
    graphBuilder.add_edge("冲咖啡", "加糖子图")
    graph = graphBuilder.compile(checkpointer=checkpointer)
    return graph


if __name__ == "__main__":
    # 1) 创建图对象
    graph = build_graph()

    # 2) 生成图片
    mk_graph(graph, os.path.splitext(os.path.basename(__file__))[0])

    # 3) 调用图
    state: State = {"水温": 58, "产物": "凉水", "咖啡固体": "咖啡豆"}
    config = {"configurable": {"thread_id": "some_id"}}
    state = graph.invoke(state, config)
    # 中断后，接收用户输入
    userDecision = input("先生您好，请问您的咖啡需要加糖吗？(是/否):")
    # 使用Command函数向graph提供用户输入的数据，继续工作流
    result = graph.invoke(Command(resume=userDecision), config=config)
    print(result)
```



## 4 工作流智能体

基于LangGraph的这种图模式，可以实现复杂的智能体架构，一般应用系统中的智能体架构有6种，分别是：

- 增强型智能体
- 链式工作流智能体
- **路由工作流智能体**
- 并行工作流智能体
- 协调器工作流智能体
- 评估优化工作流智能体

下面，我们一起来了解下这6种架构模式，重点要关注学习：**路由工作流智能体**。

### 4.1 智能体架构模式

#### 4.1.1 增强型智能体

增强型智能体的基本构建块是一个增强的LLM，其中包含检索、工具和记忆等增强功能。

![fzlzbuz3rodsmaxxqdjck1uqnbe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fzlzbuz3rodsmaxxqdjck1uqnbe.png)
> 图注：图片展示了增强型智能体的架构模式。左侧“In”为输入端，右侧“Out”为输出端。中间是LLM（大模型），与检索（Retrieval）、工具（Tools）、记忆（Memory）三个模块有交互。检索模块与LLM双向箭头，表示查询/结果；工具模块与LLM双向箭头，表示调用/响应；记忆模块与LLM单向箭头，表示读/写。该图直观呈现了增强型智能体中LLM与各增强功能模块的交互关系，与上下文对增强型智能体架构的描述相契合。

**📚 模式说明：**

- **输入输出系统**：整个流程是线性的，就是从输入到大模型（LLM），再到输出。简单来说，就是 Input → LLM → Output。
- **大模型（LLM）**：它就像个中央处理器，负责协调三个主要的功能模块——检索（Retrieval）、工具（Tools）和记忆（Memory），一起来完成信息处理。
- **这种模型，适用于业务不是很复杂的场景。**

#### 4.1.2 链式工作流智能体

工作流智能体模式，就是将任务分解为一系列步骤，其中每个LLM调用处理上一个步骤的输出，通过多步骤LLM调用分阶段处理复杂任务。

![gmdabuibmoes8bxwsoocpxslntf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gmdabuibmoes8bxwsoocpxslntf.png)
> 图注：这张图展示了链式工作流智能体的流程结构，是对应“顺序分解+中间检查（Gate）”模式的直观呈现。输入“(In)”首先进入第一个LLM调用（LLM Call 1）生成输出1，之后到达中间的Gate决策点；若Gate验证结果为“Pass”，则流程进入后续的LLM Call 2、LLM Call 3环节，最终输出结果“Out”；若Gate验证结果为“Fail”，则直接触发“Exit”环节。该图清晰对应文档中关于链式工作流智能体的“顺序分解”与“中间检查（Gate）”的规则说明，体现了工作流智能体多步骤处理复杂任务且可在中间节点阻断错误的特点。

📚 **模式说明：**

- **顺序分解：**将任务拆解为 **LLM Call 1** → **LLM Call 2** → **LLM Call 3** 的固定步骤链。
- **中间检查（Gate）：**在第一步输出后插入 **Gate** 决策点，实现中间验证阻断错误传播。
- **上下文传递：Output 1** 作为 **LLM Call 2** 的输入，**Output 2** 作为 **LLM Call 3** 的输入，通过链式传递上下文保持任务连贯性。
- **这种模式适用于任务比较复杂，但处理流程固定的场景比如：写文章 → 生成大纲 → 校验大纲 → 依据大纲编写内容 → 对内容校验 → 最后输出。**

#### 4.1.3 路由工作流智能体

路由工作流智能体，这种模式是将输入通过 **LLM Call Router** 对意图识别，再交由下游的 **LLM** 执行。

![s0bfb5tu4omkttx7pe2cmofmnsf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/s0bfb5tu4omkttx7pe2cmofmnsf.png)
> 图注：图片展示了路由工作流智能体的架构模式。输入（In）经LLM Call Router节点处理，根据输入特征决定调用LLM Call 1、LLM Call 2或LLM Call 3中的一个。若调用LLM Call 1，则其结果经LLM Call 1箭头指向Out；若调用LLM Call 2或LLM Call 3，则其结果经LLM Call 2或LLM Call 3箭头指向Out。

📚 **模式说明：**

- **动态路径选择**：Router 节点根据输入特征（如内容类型、用户意图）决定调用哪个LLM。
- **集中式决策**：所有输入需先经过Router（而非直接调用LLM），使用路由控制器降低业务系统之间的耦合度。
- **可扩展性**：图中预设3个LLM调用路径，实际上，可扩展更多分支，也就是说，路由模式可以灵活的增减处理模块，比较灵活。
- **这种模式适用于复杂业务，并且后续的处理逻辑比较独立的场景。比如：天机AI助理，推荐课程、查询课程、购买课程，这都是独立的业务，可以用独立的智能体实现。**

#### 4.1.4 并行工作流智能体

并行工作智能体，是指一个输入同时交给多个LLM去执行，再将这些大模型的输出进行汇总处理，再输出。

![liu9bxu4aox6afxjvjpctlr0nwb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/liu9bxu4aox6afxjvjpctlr0nwb.png)
> 图注：这张图展示了并行工作流智能体的运行机制，输入（In）会同时分发到三个独立的LLM Call模块：LLM Call 1、LLM Call 2和LLM Call 3，各模块并行执行处理无直接依赖关系。三个LLM的输出会汇总至Aggregator模块，经该模块整合后，最终生成输出（Out）。这一设计对应任务并行拆分与结果聚合的核心逻辑，通过并发处理提升效率，借助冗余计算增强结果可靠性，同时模块化的隔离设计还能降低单点故障的风险。

📚 **模式说明：**

- **任务并行拆分**：输入（In）同时分发给 **LLM Call 1/2/3** 并行处理，通过并发执行独立子任务提升效率。
- **结果聚合策略**：通过 **Aggregator** 模块整合多个LLM输出，用冗余计算换取结果可靠性。
- **模块化隔离**：各LLM节点无直接依赖关系（仅通过Aggregator连接），降低单点故障风险的容错设计。
- **这种模式，一般会在两种场景中使用：**将一个任务，拆分成多个子任务，并行执行，提升效率。将同一个任务，由不同的大模型执行多次，得到不同的输出，再聚合处理，以得到更准确的结果。

  例如：需要开发一个**多维度内容审核系统**，包括：是否含攻击性言论、关键数据是否准确、是否引用未授权内容等检测，这些检测可以并行执行，提升系统效率。

  例如：如医疗诊断辅助、金融风险评估



#### 4.1.5 协调器工作流智能体

协调器工作流智能体，这种模式是，由`Orchestrator LLM`作为智能调度中心，**动态生成**子任务列表，子任务可以是并行或串行执行，结果由`Synthesizer`进行聚合输出。

![gwqsbiavxoirryxev3lclra5ncd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gwqsbiavxoirryxev3lclra5ncd.png)
> 图注：图片展示了协调器工作流智能体的架构模式。输入（In）经Orchestrator处理后，动态生成子任务列表，包括LLM Call 1、LLM Call 2和LLM Call 3，这些子任务可并行或串行执行。执行结果由Synthesizer进行聚合输出，最终输出（Out）。该图直观呈现了文档中所描述的由Orchestrator作为智能调度中心，动态生成子任务列表，子任务并行或串行执行，结果由Synthesizer聚合输出的工作流智能体架构模式。

📚 **模式说明：**

- **动态任务分解：**Orchestrator 担任智能调度的核心角色，能够利用大型语言模型（LLM）动态地生成子任务列表。
- **异构模型协同工作：**通过并行或串行方式调用多个大型语言模型，并依据各个子任务的具体需求选择最适合的模型执行，以此实现高效的资源分配策略。
- **结果智能化整合：**Synthesizer 采用语义融合技术，确保来自不同模型的结果在最终输出时达到高度的一致性和连贯性。
- **这种工作流程适合处理复杂且细节不确定的任务（如编程时根据实际情况修改文件）。它看起来像并行处理，但更灵活：任务不是预先定义好的，而是由协调器根据进展动态分配和调整。**

![xvwvb5yqgooh49xmqkhcbrmtnjh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xvwvb5yqgooh49xmqkhcbrmtnjh.png)
> 图注：这张图以商业研究报告生成为例，展示了评估优化工作流智能体的完整执行流程。首先是输入环节，用户输入“分析2024年AI芯片市场趋势”；接着由协调器Orchestrator完成任务拆解与调度，分解出数据收集、竞品分析、风险预测三个子任务，调度数据专家、分析模型、风险模型三类大语言模型分别执行对应子任务；之后通过合成器Synthesizer整合三方输出、消除数据矛盾，再按企业模板重写为PPT大纲；最终输出结构化的30页研究报告。

#### 4.1.6 评估优化工作流智能体

评估优化工作流智能体，是这一种 生成 → 评估 → 反馈 循环反馈的机制。

![rwhwbgwnrouy1vxbxcfcqvibngf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rwhwbgwnrouy1vxbxcfcqvibngf.png)
> 图注：图片展示了协调器工作流智能体的架构模式。左侧输入（In）经LLM Call Generator生成解决方案（Solution），再由LLM Call Evaluator评估，若评估结果为Accepted（认可），则输出（Out）；若为Rejected+Feedback（拒绝+反馈），则返回LLM Call Generator重新生成。

📚 **模式说明：**

- **生成阶段**（Generator）：产生初始解决方案（Solution）
- **评估阶段**（Evaluator）：验证输出质量，可以选择Accepted（认可）或 Rejected+Feedback（拒绝+反馈）
- **决策阶段**：根据评估结果选择终结输出或重新生成
- **举例：**

![wjtdb9kpnopelbxljq7cappbncc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wjtdb9kpnopelbxljq7cappbncc.png)
> 图注：图片以学术摘要生成场景为例，展示了协调器工作流智能体的流程。首先，用户输入论文主题，Generator调用LLM生成初始摘要；Evaluator检查摘要是否包含研究目标、方法论、关键结论，检测到未明确说明数据来源则Rejected；Evaluator向Generator发送指令补充数据采集方法描述；Generator基于反馈生成包含数据来源的Solution；Evaluator验证通过后，流程结束。该图直观呈现了协调器工作流智能体的生成、评估、决策等环节，与上下文对协调器工作流智能体模式说明相呼应。

**场景：**

在文学翻译领域，译者型语言模型（LLM）可能在初次尝试时未能完全把握文本中的细微差异，但评估型语言模型（LLM）能够对此提供具有建设性的反馈

在执行复杂的搜索任务时，往往需要通过多轮的搜索与分析来确保信息收集的全面性

在此过程中，评估者将根据已获取的信息质量及完整性，判断是否有必要展开进一步的搜索活动

#### 4.1.7 总结对比

<sheet sheet-id="ZDEKBz" token="MKH2sh0oeh6FXRtIzN7cxmcOng5"></sheet>

**模式选型建议，根据业务需求选择：**

- **简单任务** → 增强型智能体 / 链式工作流智能体
- **多分支处理** → 路由模式
- **高实时性** → 并行化（需任务可拆分）
- **超复杂任务** → 协调器工作流智能体
- **超高可靠性** → 评估优化工作流智能体

### 4.2 路由工作流智能体

路由工作流智能体的架构，一般是解决复杂的业务场景，例如：在线客服，用户提问后，会按照问题进行分类，然后将请求路由到定价、退款、退货等特定流程

![quazba04zopjyvxm40nc2ih5nsd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/quazba04zopjyvxm40nc2ih5nsd.png)
> 图注：这张图展示了路由工作流智能体的架构，对应文档中要实现的根据用户输入选择生成故事、笑话或诗歌的智能体助手架构。图中输入“In”进入标有“Router”的路由组件，该路由组件通过三条带箭头的虚线，将请求分别指向三个功能组件，每个功能组件配有脑状图标，最终每个组件对应输出“Out”，体现了路由智能体分类请求并将其分发到对应特定功能流程的核心逻辑，和文档中路由工作流智能体用于分类请求并定向处理的内容相契合。

我们将实现一个智能体助手，根据用户的输入来决定是生成故事、笑话、诗歌其中的一个，架构如下：

![xhr6b6kutognuvxkld1cyjolnne](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xhr6b6kutognuvxkld1cyjolnne.png)
> 图注：图片展示了智能体助手的架构图。用户输入经路由智能体处理后，输出“story”“joke”“poem”三种类型，分别对应故事生成Agent、笑话生成Agent、诗歌生成Agent。这些生成Agent将内容传至最终输出，完成智能体助手的执行层功能。该图与上下文紧密相关，直观呈现了根据用户输入决定生成故事、笑话或诗歌这一工作流智能体架构。

代码实现

```Python
import os
import getpass
from typing import TypedDict

from langchain.chat_models import init_chat_model
from typing_extensions import Literal
from pydantic import BaseModel, Field
from langchain.messages import HumanMessage, SystemMessage
from langgraph.graph import StateGraph, START, END

# 定义状态类
class AgentState(TypedDict):
    input: str  # 用户输入
    decision: str  # 路由决策结果
    output: str  # 最终生成内容


# 初始化大模型
llm = init_chat_model(
    model="qwen3-max",
    model_provider="openai",
    api_key=os.getenv("OPENAI_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)


# 调用大模型的通用方法
def generate_text(state: AgentState, task_desc: str):
    print(f"[Worker] 当前任务：{task_desc}")
    print("[Worker] 开始调用大模型生成内容...")
    prompt = f"{task_desc}\n\n用户请求：{state['input']}"
    result = llm.invoke(prompt)
    print("[Worker] 内容生成完成")
    return {"output": result.content}


# 路由结构定义（核心）
class Route(BaseModel):
    step: Literal["story", "joke", "poem"] = Field(
        description="根据用户输入，选择下一步执行的节点"
    )
router_llm = llm.with_structured_output(Route)


# 1) 定义节点
def write_story(state: AgentState):
    print("[Worker] 进入 write_story 节点")
    return generate_text(state, "请写一个故事")


def write_joke(state: AgentState):
    print("[Worker] 进入 write_joke 节点")
    return generate_text(state, "请写一个笑话")


def write_poem(state: AgentState):
    print("[Worker] 进入 write_poem 节点")
    return generate_text(state, "请写一首诗")


def router_node(state: AgentState):
    print(f"\n[Router] 接收到用户输入:{state['input']}")
    decision = router_llm.invoke([
        SystemMessage(content="根据用户请求，将任务路由到 story、joke 或 poem"),
        HumanMessage(content=state["input"])
    ])
    print(f"[Router] 模型返回的结构化决策：step = {decision.step}")
    return {"decision": decision.step}


# 2) 定义条件边
def route_selector(state: AgentState):
    print(f"[Selector] 开始条件判断,decision = {state['decision']}")
    if state["decision"] == "story":
        print("[Selector] 路由到 write_story")
        return "write_story"
    elif state["decision"] == "joke":
        print("[Selector] 路由到 write_joke")
        return "write_joke"
    elif state["decision"] == "poem":
        print("[Selector] 路由到 write_poem")
        return "write_poem"

# 3) 构建图
def build_graph():
    builder = StateGraph(AgentState)

    # 注册节点
    builder.add_node("router", router_node)
    builder.add_node("write_story", write_story)
    builder.add_node("write_joke", write_joke)
    builder.add_node("write_poem", write_poem)

    # 注册表
    builder.add_edge(START, "router")
    # 条件分支
    builder.add_conditional_edges(
        "router",
        route_selector,
        {
            "write_story": "write_story",
            "write_joke": "write_joke",
            "write_poem": "write_poem",
        }
    )
    builder.add_edge("write_story", END)
    builder.add_edge("write_joke", END)
    builder.add_edge("write_poem", END)

    # 编译工作流
    graph = builder.compile()
    return graph


if __name__ == "__main__":
    graph = build_graph()
    print("🚀 路由智能体已启动（输入 exit 退出）")
    while True:
        try:
            user_input = input("\n👤 用户输入: ")
            if user_input.lower() == "exit":
                print("👋 程序结束")
                break

            print("\n🔍 ===== 开始执行工作流 =====")
            result = graph.invoke({"input": user_input})
            print("🔍 ===== 工作流执行结束 =====\n")

            print("🤖 Assistant 输出：")
            print(result["output"])
        except Exception as e:
            print("❌ 发生错误：" + str(e))
```

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / LangChain 与 LangGraph / 09-LangGraph · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/RZYqwKrNAiFkeNkfWerc4jVDnzo)
