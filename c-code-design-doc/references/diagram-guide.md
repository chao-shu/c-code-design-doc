# PlantUML 建模与绘图规范

用于“从 C 源码学习设计”的文档。目标是忠实表达责任、边界、接口和行为，读者能顺着图回到源码。

绘图时同时执行 [专业工程图视觉规范](visual-style.md)，将统一基础样式内嵌到每个独立图块，并按其中的通过条件验收。

## 选择图型与信息层次

- **上下文**：目标文件与直接调用方/依赖，先让读者知道模块位于哪里。
- **逻辑**：组件/接口图表达职责分工；C 的结构体、枚举和函数指针表可以在类图中标注 `<<struct>>`、`<<enum>>`、`<<function_pointer_table>>`。这只是数据建模记法，不表示 C 有类方法或继承。
- **开发**：包、文件/制品与依赖图，说明源码组织、包含与构建；构建图中的转换箭头明确标为“编译输入”“链接输入”，不冒充 UML 标准关系。
- **进程**：带执行上下文的交互图、活动图或上下文关系图。线程/任务与文件是不同维度，同一个文件可以在多个上下文执行。
- **物理**：部署图展示节点、执行环境与制品；节点需要实际配置或代码支持，产物被编译出来不代表已经部署到某设备。
- **场景**：时序图表达具体请求和分支；用例图仅在多个外部角色及目标确实需要归纳时增加。
- **状态**：只有枚举、分支、赋值及生命周期支持真实状态机时才画。初始化成功不一定意味着存在名为 READY 的状态。

不强制每类图各一张。可简化很小的模块，但五个视图的解释必须覆盖；证据不足时说明，不补造图。

## 专业图示的约束

1. **单图回答一个问题**。不要把源码包含、运行调用、线程调度和物理网络混成一个总图。
2. **层次一致**。同层参与者使用相近粒度；外部依赖、目标模块和环境边界用分组或构造型区分。逻辑组件上的文件名是可追溯标识，不表示文件独立部署。
3. **每条关键边可回溯**。图后附证据范围，必要时用小表列出“关系 → E 编号”。代码注释里的证据不能替代读者可见的图后说明。
4. **使用实际名称**。节点标签保留文件名，边写真实函数/事件/接口；不要只写“调用”“处理”“数据”。过长签名放接口表，图中保留决定语义的参数。
5. **方向有语义**。依赖从使用方指向被依赖方，数据流按实际方向，时序按真实先后。接口提供者与使用者不能画反。UML interface 节点命名为操作契约（例如 Session API），声明来源另标 `session.h`；头文件制品本身放在开发视图，不把任意头文件自动当作一个 UML 接口。
6. **未知显式表达**。无法解析的回调可画为“未解析回调目标”，标明 `<<unresolved>>` 并附绑定缺口。示意性分组要注明，不画成已确认的架构实体。
7. **克制布局**。通常单图约 5–9 个主要元素最易读；复杂图按场景或边界拆分，此数量只是阅读建议。结构图可横向或纵向，时序图按时间向下展开。图过宽时先缩短接口标签、把完整签名移到接口表，再调整方向或拆图；不要以缩小字号掩盖拥挤。只使用有限颜色。
8. **共用命名**。同一文件、接口、队列和状态在全部图中保持一致名称。跨图可用不同别名，但同一代码块内别名必须唯一。

## 通用语法与兼容性

每个图使用 `plantuml` 围栏，包含 `@startuml` / `@enduml`；不嵌套 Markdown 围栏。图块独立渲染，不依赖其他图块定义。

使用基础 PlantUML 语法，显式声明节点和参与者。标签加双引号，标识符用简单 ASCII 别名，如 `M_SESSION`；显示标签中用 `\n` 换行。保持中文说明，文件名/符号使用原名。

开发图的源码包含边可写 `include`，在图例说明其对应 `#include`；源码原文保留在 C 代码块。避免将带格式标记的源码片段直接塞进关系标签后，未经预览就认定显示结果与原文一致。

每个图块采用 visual-style.md 的白底蓝灰样式；按图型选择方向、`hide footbox` 和 `autonumber`，不照搬无关图型设置。最终以语义和视觉两项验收结果为准。

不使用远程 `!include`、`!includeurl`、外部图片、C4 宏库、需要插件的图标或未知主题。这样图代码更容易在不同免费渲染器中使用；仍不能保证任意插件支持全部语法。

## 常用关系

| 图型 | 表达 | 含义/注意点 |
|---|---|---|
| 组件/开发 | `A ..> B : ...` | 使用/依赖；写清是声明依赖、直接调用依赖还是具体构建关系 |
| 组件 | `Provider -- I` | 提供接口（棒棒糖形式）；`Consumer ..> I` 表示使用 |
| 数据结构 | `A --> B : pointer / borrows` | 导航/引用；指针不等于拥有 |
| 数据结构 | `A *-- B` | 仅在独占包含及生命周期约束有证据时用组合 |
| 时序 | `A -> B : function()` | 同步调用 |
| 时序 | `A --> B : result` | 返回；不是异步消息 |
| 时序 | `A ->> B : event` | 已确认的异步消息；队列传递应表现入队与消费边界 |
| 状态 | `S1 --> S2 : event [guard] / action` | 事件、守卫条件和动作必须对应实际实现 |

箭头风格按各图型的语法解释，不能把时序图中的 `-->` 机械地用于其他图型并赋予“返回”含义。

### 时序图细则

- 按主链从左到右排列参与者，显式声明每个参与者及其文件来源。
- 一个参与者代表文件时，在标题/说明注明“参与者按文件分组”；执行线程另用 `box` 或 note 说明。不能从参与者数量推导线程数量。
- 用 `alt/else` 标真实分支，用 `opt` 标可选行为，用 `loop` 标真实循环；并发证据存在才用 `par`。
- 真实函数指针调用要标出指针成员和已确认目标；直接在当前调用栈执行的回调仍使用同步箭头。
- 调用消息写操作名和关键输入，返回写关键结果/错误。不要画无意义的每层返回，也不能漏掉改变结果的错误转换。
- 激活条不是装饰；使用时配对平衡。若分支让激活难以准确表达，可以省略激活条，不能为视觉效果伪造执行区间。
- “成功示例调用链”不能隐藏初始化前提或配置分支；错误路径应表现已经分配资源的释放和最终状态。

## 完整示例

以下全部是**虚构的建模与样式演示**，不存在对应仓库证据。不得原样混入真实文档；使用时替换所有文件、符号、关系和说明。真实文档中每图后要有依据。

### 逻辑视图：文件与接口

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
skinparam componentStyle uml2
skinparam componentBackgroundColor #F8FAFC
skinparam componentBorderColor #64748B
skinparam interfaceBackgroundColor #FFFFFF
skinparam interfaceBorderColor #475569
skinparam nodesep 32
skinparam ranksep 36
title 图 L1｜文件职责与接口（逻辑视图）
left to right direction
component "app/main.c\n调用方" as M_APP
component "src/session.c\n目标模块 · 会话状态" as M_SESSION #EFF6FF
interface "Session API\nsession_start()" as I_SESSION
interface "Transport API\ntransport_send()" as I_TRANSPORT
component "src/transport.c\n传输适配" as M_TRANSPORT
M_APP ..> I_SESSION : session_start()
M_SESSION -- I_SESSION
M_SESSION ..> I_TRANSPORT : transport_send()
M_TRANSPORT -- I_TRANSPORT
legend bottom
浅蓝：目标模块；灰白：调用方与依赖
API 声明来源：session.h、transport.h
endlegend
@enduml
```

### 开发视图：包含与构建输入

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
skinparam artifactBackgroundColor #F8FAFC
skinparam artifactBorderColor #64748B
skinparam nodesep 32
skinparam ranksep 36
title 图 D1｜包含与构建输入（开发视图）
left to right direction
artifact "src/session.c\n目标文件" as F_SESSION #EFF6FF
artifact "include/session.h" as F_SESSION_H
artifact "include/transport.h" as F_TRANSPORT_H
artifact "src/transport.c" as F_TRANSPORT
artifact "session.o" as O_SESSION
artifact "transport.o" as O_TRANSPORT
artifact "libdemo.a" as A_LIB
F_SESSION ..> F_SESSION_H : include
F_SESSION ..> F_TRANSPORT_H : include
F_TRANSPORT ..> F_TRANSPORT_H : include
F_SESSION --> O_SESSION : 编译输入
F_TRANSPORT --> O_TRANSPORT : 编译输入
O_SESSION --> A_LIB : 归档输入
O_TRANSPORT --> A_LIB : 归档输入
legend bottom
虚线：源码包含依赖
实线：构建输入关系（本图约定）
endlegend
@enduml
```

### 进程视图：有证据时才使用任务与队列

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
skinparam rectangleBackgroundColor #F8FAFC
skinparam rectangleBorderColor #64748B
skinparam queueBackgroundColor #F8FAFC
skinparam queueBorderColor #64748B
skinparam nodesep 32
skinparam ranksep 36
title 图 P1｜任务与消息边界（进程视图）
left to right direction
rectangle "设备进程" as P_DEVICE <<process>> #FFFFFF {
  rectangle "接收任务\nrx_task()" as T_RX <<thread>>
  queue "frame_queue\n有界队列" as Q_FRAME
  rectangle "处理任务（关注上下文）\nworker_task()" as T_WORKER <<thread>> #EFF6FF
}
T_RX --> Q_FRAME : 入队 frame\nqueue_put(frame)
Q_FRAME --> T_WORKER : 出队 frame
note bottom of T_WORKER
worker_task() 调用 queue_get()，
取出 frame 后同步执行
session_process(frame)。
end note
legend bottom
箭头表示消息投递/消费方向，
不是直接函数调用关系。
endlegend
@enduml
```

### 物理视图：节点、执行环境与制品

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
skinparam nodeBackgroundColor #F8FAFC
skinparam nodeBorderColor #64748B
skinparam artifactBackgroundColor #FFFFFF
skinparam artifactBorderColor #64748B
skinparam nodesep 32
skinparam ranksep 40
title 图 H1｜制品与运行节点（物理视图）
left to right direction
node "目标设备" as N_TARGET <<device>> {
  node "RTOS" as E_RTOS <<executionEnvironment>> {
    artifact "demo.elf\n关注制品" as A_DEMO #EFF6FF
  }
}
node "对端设备" as N_PEER <<device>>
N_TARGET -- N_PEER : UART
@enduml
```

### 场景视图：同步回调与错误传播

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
skinparam sequenceParticipantBackgroundColor #F8FAFC
skinparam sequenceParticipantBorderColor #64748B
skinparam sequenceLifeLineBorderColor #94A3B8
skinparam sequenceLifeLineBackgroundColor #EFF6FF
skinparam sequenceArrowColor #475569
skinparam sequenceArrowFontSize 13
skinparam sequenceGroupBackgroundColor #F8FAFC
skinparam sequenceGroupBodyBackgroundColor #FFFFFF
skinparam sequenceGroupBorderColor #94A3B8
skinparam sequenceGroupFontColor #0F172A
skinparam sequenceMessageAlign left
title 图 S1｜发送与同步回调（场景视图）
hide footbox
autonumber
participant "app/main.c" as APP
participant "src/session.c\n目标模块" as SESSION #EFF6FF
participant "src/transport.c" as TRANSPORT
note over APP, TRANSPORT
前提：回调已注册；参与者按文件分组。
调用与回调均在调用方上下文同步执行。
end note
APP -> SESSION : session_send(data, len)
SESSION -> TRANSPORT : transport_send(data, len)
alt 传输失败
  TRANSPORT --> SESSION : ERR_IO
  SESSION --> APP : ERR_IO
else 传输成功
  TRANSPORT --> SESSION : OK
  SESSION -> APP : on_done(ctx, OK)\n已绑定 app_done()
  APP --> SESSION : return
  SESSION --> APP : OK
end
@enduml
```

### 状态图：只有源码具备状态机时使用

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
hide empty description
skinparam stateBackgroundColor #F8FAFC
skinparam stateBorderColor #64748B
skinparam stateStartColor #334155
skinparam stateEndColor #334155
skinparam nodesep 36
skinparam ranksep 40
title 图 T1｜资源状态与迁移
state "CLOSED" as S_CLOSED
state "READY" as S_READY
state "BUSY" as S_BUSY
[*] -down-> S_CLOSED
S_CLOSED -down-> S_READY : open() [成功]\n/ 保存句柄
S_READY -down-> S_BUSY : submit() [参数有效]
S_BUSY --> S_READY : complete()\n/ 清除在途标记
S_READY --> S_CLOSED : close()\n/ 释放句柄
@enduml
```

### C 数据结构图：引用与所有权分开

```plantuml
@startuml
skinparam backgroundColor #FFFFFF
skinparam shadowing false
skinparam defaultFontName "Microsoft YaHei"
skinparam defaultFontSize 14
skinparam defaultFontColor #0F172A
skinparam titleFontColor #0F172A
skinparam titleFontSize 18
skinparam ArrowColor #475569
skinparam ArrowFontColor #334155
skinparam ArrowFontSize 13
skinparam ArrowThickness 1
skinparam NoteBackgroundColor #F8FAFC
skinparam NoteBorderColor #CBD5E1
skinparam NoteFontColor #475569
skinparam NoteFontSize 13
skinparam LegendBackgroundColor #FFFFFF
skinparam LegendBorderColor #FFFFFF
skinparam LegendFontColor #475569
skinparam LegendFontSize 12
hide circle
hide empty members
skinparam classAttributeIconSize 0
skinparam classBackgroundColor #F8FAFC
skinparam classBorderColor #64748B
skinparam nodesep 32
skinparam ranksep 36
title 图 T2｜结构体与借用关系
class "session_t" as D_SESSION <<struct>> #EFF6FF {
  const transport_ops_t *ops
  void *user_ctx
}
class "transport_ops_t" as D_OPS <<function_pointer_table>> {
  {field} int (*send)(const void *data, size_t len)
}
class "app_context_t" as D_CTX <<struct>>
D_SESSION --> D_OPS : ops / borrows
D_SESSION --> D_CTX : user_ctx / borrows
note bottom of D_SESSION
目标结构：session_t。
ops 与 user_ctx 由调用方持有，
其有效期须覆盖本结构的使用期。
end note
@enduml
```

## 验证与取舍

先检查图与源码语义一致，再检查图语法和布局；“能渲染”不能证明内容准确。

本地已有 PlantUML JAR 时，可按现有版本 CLI 检查临时提取的图。例如在支持传统选项的版本中：

```text
java -jar <本地plantuml.jar> -checkonly <临时图目录>
java -jar <本地plantuml.jar> -tsvg <临时图目录>
```

以真实退出状态和输出为准。图形布局可能还依赖 Graphviz 或版本自带的布局能力；缺依赖时不要将失败说成通过，也不把语法检查说成视觉检查。

有预览能力时检查交叉线、遮挡、长标签、中文字体和错误页；没有预览能力则写明未进行视觉检查。最多按实际错误做必要修复，不能以换掉真实关系来换取渲染成功。

语法资料（需要核对时查阅，不是输出依赖）：

- [PlantUML 组件图](https://plantuml.com/component-diagram)
- [PlantUML 时序图](https://plantuml.com/sequence-diagram)
- [PlantUML 部署图](https://plantuml.com/deployment-diagram)
- [PlantUML 状态图](https://plantuml.com/state-diagram)
- [PlantUML 类图](https://plantuml.com/class-diagram)
- [4+1 原始论文](https://www.cs.ubc.ca/~gregor/teaching/papers/4%2B1view-architecture.pdf)
