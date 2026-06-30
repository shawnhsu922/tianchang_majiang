# 天长麻将微信小程序（Tianchang Mahjong Mini Program）

# 04_Rule_Cases

Version：1.0.0  
Status：Draft  
Author：Shawn Hsu & ChatGPT

---

# 一、文档目的

本文档用于定义服务器规则测试案例。

所有案例均按照服务器 Action 组织，未来 Codex 必须根据本文档编写自动化测试。

---

# 二、测试输入说明

本文档中的手牌、摸牌、牌墙状态均为“构造测试输入”。

它们不要求来自真实牌局发牌结果，但必须满足：

1. 牌张数量合法。
2. 单张牌最多出现四张。
3. 测试场景可以被程序稳定复现。
4. 预期结果必须明确。
5. 花数必须对应 `03_Scoring_Table.md`。

本文档禁止使用以下模糊描述：

- 其他牌组
- 类似情况
- 约等于
- 大概可以胡
- 理论上

---

# 三、服务器 Action 列表

| Action | 含义 |
|--------|------|
| CREATE_ROOM | 创建房间 |
| JOIN_ROOM | 加入房间 |
| READY | 玩家准备 |
| START_GAME | 开始游戏 |
| DRAW | 摸牌 |
| DISCARD | 出牌 |
| PENG | 碰牌 |
| MING_GANG | 明杠 |
| BU_GANG | 补杠 |
| AN_GANG | 暗杠 |
| BU_HUA | 补花 |
| TING | 报听 |
| WIN | 胡牌 |
| PASS | 放弃操作 |
| TRUSTEE | 托管 |
| RECONNECT | 重连 |
| SETTLEMENT | 结算 |
| NEXT_ROUND | 下一局 |

---

# 四、Rule Case 格式

```text
RC-模块-编号

Title：
测试名称

Related Rules：
关联规则

Action：
服务器动作

Precondition：
前置条件

Input：
输入数据

Expected：
预期输出

Score：
预期花数，如无则为 N/A

Broadcast：
服务器应广播内容

Notes：
说明
```

---

# 五、DRAW 测试案例

## RC-DRAW-001 普通摸牌

Title：

普通摸牌后进入出牌状态

Related Rules：

- GR-013

Action：

DRAW

Precondition：

- 游戏已开始。
- 当前轮到东家摸牌。
- 东家未报听。
- 摸牌不是基本花。
- 摸牌后不能胡牌。
- 摸牌后没有杠牌机会。

Input：

当前玩家：

东家

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
9筒
```

Expected：

- 东家手牌增加 9筒。
- 不触发胡牌。
- 不触发补花。
- 不触发杠牌。
- 服务器进入等待东家出牌状态。

Score：

N/A

Broadcast：

```text
DRAW_SUCCESS
WAIT_DISCARD
```

Notes：

普通摸牌流程。

---

## RC-DRAW-002 摸到基本花

Title：

摸到基本花后进入补花流程

Related Rules：

- GR-010
- GR-013

Action：

DRAW

Precondition：

- 游戏已开始。
- 当前轮到东家摸牌。
- 摸到牌为基本花。

Input：

当前玩家：

东家

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
东
```

Expected：

- 东家获得 1 张基本花。
- 东家不得出牌。
- 服务器进入补花流程。
- 需要从杠尾继续补牌。

Score：

```text
基本花 +1
```

Broadcast：

```text
DRAW_FLOWER
WAIT_BU_HUA
```

Notes：

东南西北中发白均属于基本花。

---

## RC-DRAW-003 摸牌后可胡牌

Title：

摸牌后形成合法胡牌

Related Rules：

- GR-039
- GR-041

Action：

DRAW

Precondition：

- 当前玩家未胡牌。
- 摸牌后形成合法普通胡。

Input：

当前玩家：

东家

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
8万
```

Expected：

- 服务器识别为可胡牌。
- 玩家可以选择胡牌。
- 玩家可以选择放弃胡牌。
- 程序不得自动胡牌。

Score：

若选择胡牌：

```text
单吊 7
```

Broadcast：

```text
CAN_WIN
WAIT_PLAYER_DECISION
```

Notes：

摸 8万 后形成四组面子加一对将。

---

## RC-DRAW-004 报听后摸到可胡牌但可放弃

Title：

报听后自摸可选择不胡

Related Rules：

- GR-036
- GR-039

Action：

DRAW

Precondition：

- 当前玩家已报听。
- 摸牌后形成合法胡牌。

Input：

当前玩家：

东家

报听：

是

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
8万
```

Expected：

- 服务器提示可以胡牌。
- 玩家可以选择胡牌。
- 玩家可以选择放弃胡牌。
- 若放弃胡牌，游戏继续。

Score：

若胡牌：

```text
单吊 7
```

Broadcast：

```text
CAN_WIN
WAIT_PLAYER_DECISION
```

Notes：

报听后摸到胡牌也不强制胡。

---

## RC-DRAW-005 摸牌后形成暗杠机会

Title：

摸牌形成暗杠但不自动杠

Related Rules：

- GR-024
- GR-025

Action：

DRAW

Precondition：

- 当前玩家摸牌后手中有四张相同牌。
- 玩家未报听。

Input：

当前玩家：

东家

当前手牌：

```text
3万 3万 3万
1条 2条 3条
4条 5条 6条
3筒 4筒 5筒
8万
```

摸牌：

```text
3万
```

Expected：

- 服务器识别暗杠机会。
- 玩家可以选择暗杠。
- 玩家可以选择不杠。
- 程序不得自动暗杠。

Score：

若选择暗杠：

```text
暗杠 +4
```

Broadcast：

```text
CAN_AN_GANG
WAIT_PLAYER_DECISION
```

Notes：

暗杠为可选操作。

---

## RC-DRAW-006 摸到最后一张并胡牌

Title：

最后一张自摸胡牌触发海底捞月

Related Rules：

- GR-050

Action：

DRAW

Precondition：

- 当前牌墙只剩最后一张有效牌。
- 当前玩家摸到该牌后胡牌。

Input：

当前玩家：

东家

牌墙剩余：

```text
1张
```

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
8万
```

Expected：

- 服务器识别可胡牌。
- 若玩家选择胡牌，事件包含海底捞月。

Score：

```text
单吊 7
海底捞月 +10
总花 17
```

Broadcast：

```text
CAN_WIN
EVENT_HAIDI
WAIT_PLAYER_DECISION
```

Notes：

最后一张正常摸牌胡牌，算海底捞月。

---

## RC-DRAW-007 杠后摸牌胡牌

Title：

杠后摸牌胡牌触发杠开

Related Rules：

- GR-049

Action：

DRAW

Precondition：

- 当前玩家刚完成杠牌。
- 本次摸牌来自杠尾。
- 摸牌后胡牌。

Input：

当前玩家：

东家

玩家状态：

杠后摸牌

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
8万
```

Expected：

- 服务器识别可胡牌。
- 若玩家选择胡牌，事件包含杠开。

Score：

```text
单吊 7
杠开 +2
总花 9
```

Broadcast：

```text
CAN_WIN
EVENT_GANGKAI
WAIT_PLAYER_DECISION
```

Notes：

杠尾摸牌胡牌，算杠开。

---

## RC-DRAW-008 海底捞月与杠开同时成立

Title：

最后一张有效牌来自杠尾且胡牌

Related Rules：

- GR-049
- GR-050

Action：

DRAW

Precondition：

- 当前玩家因补花或杠牌从杠尾摸牌。
- 摸到的是本局最后一张有效牌。
- 摸牌后胡牌。

Input：

当前玩家：

东家

玩家状态：

杠后摸牌

牌墙剩余：

```text
最后一张有效牌
```

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万
```

摸牌：

```text
8万
```

Expected：

- 服务器识别可胡牌。
- 事件同时包含海底捞月和杠开。
- 两项事件奖励允许叠加。

Score：

```text
单吊 7
海底捞月 +10
杠开 +2
总花 19
```

Broadcast：

```text
CAN_WIN
EVENT_HAIDI
EVENT_GANGKAI
WAIT_PLAYER_DECISION
```

---

## RC-EVENT-001 天胡事件互斥验证

Title：

天胡不得与其他胡牌事件同时成立。

Related Rules：

- GR-051

Action：

WIN

Precondition：

玩家满足天胡条件。

Input：

玩家状态：

```text
天胡
```

Expected：

Pattern：

正常计算。

Event：

```text
TIAN_HU
```

不得包含：

```text
GANG_KAI

HAI_DI
```

Flower Calculation：

```text
牌型花

+

天胡200
```

Broadcast：

```text
EVENT_TIANHU
```

Notes：

用于验证天胡事件具有最高优先级，与其他胡牌事件互斥。

---

## RC-EVENT-002 海底捞月与杠开可同时成立

Title：

海底捞月与杠开允许叠加。

Related Rules：

- GR-049
- GR-050
- GR-051

Action：

WIN

Precondition：

玩家因杠牌或补花，

摸到本局最后一张有效牌，

并完成胡牌。

Expected：

Pattern：

正常计算。

Event：

```text
GANG_KAI

HAI_DI
```

Flower Calculation：

```text
牌型花

+

海底捞月10

+

杠开2
```

Broadcast：

```text
EVENT_GANGKAI

EVENT_HAIDI
```

Notes：

用于验证海底捞月与杠开允许同时成立，并同时计入花数。

Notes：

这是回归测试案例，防止海底与杠开被错误设为互斥。

---

# 六、DISCARD 测试案例

## RC-DISCARD-001 正常出牌

Title：

玩家正常打出一张手牌。

Related Rules：

- GR-014

Action：

DISCARD

Precondition：

- 当前玩家已摸牌。
- 当前玩家未胡牌。
- 当前玩家处于等待出牌状态。

Input：

当前玩家：

东家

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万 9筒
```

出牌：

```text
9筒
```

Expected：

- 9筒 从东家手牌中移除。
- 9筒 显示在弃牌区。
- 服务器广播出牌结果。
- 进入其他玩家响应阶段。

Broadcast：

```text
DISCARD_SUCCESS
WAIT_RESPONSE
```

Notes：

普通出牌流程。

---

## RC-DISCARD-002 报听后出牌保持听牌

Title：

报听后只能打出不破坏听牌状态的牌。

Related Rules：

- GR-032
- GR-034

Action：

DISCARD

Precondition：

- 当前玩家已报听。
- 玩家出牌后仍保持听牌。

Input：

当前玩家：

东家

报听：

是

当前手牌：

```text
1万 2万 3万
2条 3条 4条
5条 6条 7条
3筒 4筒 5筒
8万 9筒
```

出牌：

```text
9筒
```

Expected：

- 允许出牌。
- 出牌后仍保持听牌状态。
- 服务器进入其他玩家响应阶段。

Broadcast：

```text
DISCARD_SUCCESS
WAIT_RESPONSE
```

Notes：

报听后不能换牌，但可以正常打出摸到的无关牌。

---

## RC-DISCARD-003 报听后禁止破坏听牌

Title：

报听后不得打出导致失去听牌的牌。

Related Rules：

- GR-032
- GR-034

Action：

DISCARD

Precondition：

- 当前玩家已报听。
- 出牌后将不再听牌。

Input：

当前玩家：

东家

报听：

是

出牌：

```text
待破坏听牌的牌
```

Expected：

- 服务器拒绝出牌。
- 玩家仍保持原手牌。
- 游戏状态不变。

Broadcast：

```text
DISCARD_FORBIDDEN_NOT_TING
```

Notes：

具体牌例后续在听牌专项案例中补充。

---

## RC-DISCARD-004 出牌后无人响应

Title：

玩家出牌后无人碰杠。

Related Rules：

- GR-014
- GR-015

Action：

DISCARD

Precondition：

- 东家打出一张牌。
- 其余三家均不能碰。
- 其余三家均不能杠。

Input：

出牌：

```text
9筒
```

Expected：

- 出牌成功。
- 无响应。
- 轮到下一家摸牌。

Broadcast：

```text
DISCARD_SUCCESS
NO_RESPONSE
NEXT_PLAYER_DRAW
```

Notes：

无人响应时直接进入下一位玩家摸牌。

---

## RC-DISCARD-005 出牌后有人可碰

Title：

玩家出牌后其他玩家可以碰。

Related Rules：

- GR-015
- GR-016

Action：

DISCARD

Precondition：

- 东家打出 5条。
- 南家手中有两张 5条。
- 南家未报听。

Input：

东家出牌：

```text
5条
```

南家手牌包含：

```text
5条 5条
```

Expected：

- 服务器识别南家可碰。
- 进入等待南家决策状态。

Broadcast：

```text
DISCARD_SUCCESS
CAN_PENG
WAIT_PLAYER_DECISION
```

Notes：

碰牌不是强制操作。

---

## RC-DISCARD-006 出牌后有人可明杠

Title：

玩家出牌后其他玩家可以明杠。

Related Rules：

- GR-022

Action：

DISCARD

Precondition：

- 东家打出 5条。
- 南家手中有三张 5条。
- 南家未报听。

Input：

东家出牌：

```text
5条
```

南家手牌包含：

```text
5条 5条 5条
```

Expected：

- 服务器识别南家可明杠。
- 进入等待南家决策状态。

Broadcast：

```text
DISCARD_SUCCESS
CAN_MING_GANG
WAIT_PLAYER_DECISION
```

Notes：

明杠不是强制操作，除非满足必须杠规则。

---

# 七、PENG 测试案例

## RC-PENG-001 正常碰牌

Title：

玩家正常碰牌成功。

Related Rules：

- GR-015
- GR-016

Action：

PENG

Precondition：

- 上一名玩家刚打出 5条。
- 当前玩家手中有两张 5条。
- 当前玩家未报听。

Input：

出牌：

```text
5条
```

当前玩家手牌包含：

```text
5条 5条
```

Expected：

- 碰牌成功。
- 三张 5条 形成公开碰牌组。
- 当前玩家获得出牌权。
- 当前玩家必须打出一张牌。

Broadcast：

```text
PENG_SUCCESS
WAIT_DISCARD
```

Notes：

碰牌后必须出牌。

---

## RC-PENG-002 没有对子不能碰

Title：

玩家手中不足两张相同牌时禁止碰牌。

Related Rules：

- GR-015

Action：

PENG

Precondition：

- 上一名玩家刚打出 5条。
- 当前玩家手中只有一张 5条。

Input：

出牌：

```text
5条
```

当前玩家手牌包含：

```text
5条
```

Expected：

- 服务器拒绝碰牌。
- 游戏状态不变。

Broadcast：

```text
PENG_FORBIDDEN
```

Notes：

必须拥有两张相同牌才能碰。

---

## RC-PENG-003 报听后禁止碰牌

Title：

玩家报听后不得碰牌。

Related Rules：

- GR-019
- GR-034

Action：

PENG

Precondition：

- 当前玩家已报听。
- 其他玩家打出的牌，当前玩家本可碰。

Input：

出牌：

```text
5条
```

当前玩家手牌包含：

```text
5条 5条
```

报听：

是

Expected：

- 服务器拒绝碰牌。
- 玩家听牌状态不变。

Broadcast：

```text
PENG_FORBIDDEN_AFTER_TING
```

Notes：

报听后只能杠，不能碰。

---

## RC-PENG-004 碰牌组不得拆分

Title：

碰牌成功后公开牌组不得拆分。

Related Rules：

- GR-016
- GR-029

Action：

PENG

Precondition：

- 玩家碰 5条 成功。

Input：

碰牌组：

```text
5条 5条 5条
```

Expected：

- 三张 5条 锁定为公开牌组。
- 不得重新进入手牌。
- 不得拆分组成顺子。

Broadcast：

```text
PENG_LOCKED
```

Notes：

碰牌组为固定牌组。

---

## RC-PENG-005 碰牌后必须出牌

Title：

碰牌后不能跳过出牌。

Related Rules：

- GR-016

Action：

PENG

Precondition：

- 玩家碰牌成功。

Input：

碰牌：

```text
5条
```

Expected：

- 玩家获得出牌权。
- 服务器进入 WAIT_DISCARD。
- 玩家必须打出一张手牌。

Broadcast：

```text
PENG_SUCCESS
WAIT_DISCARD
```

Notes：

碰牌不会直接结束回合。

---

---

# 八、MING_GANG 测试案例

## RC-MING-GANG-001 正常明杠

Title：

玩家正常明杠。

Related Rules：

- GR-022

Action：

MING_GANG

Precondition：

- 上家打出一张牌。
- 当前玩家手中已有三张相同牌。
- 当前玩家未报听。

Input：

上家出牌：

```text
8万
```

当前玩家手牌：

```text
8万
8万
8万
```

Expected：

- 明杠成功。
- 四张牌移动至公开区域。
- 当前玩家获得杠牌奖励。
- 从杠尾继续摸牌。

Pattern：

N/A

Events：

杠牌

Flower Calculation：

```text
明杠 +2
```

Broadcast：

```text
MING_GANG_SUCCESS

DRAW_FROM_GANG
```

Notes：

标准明杠流程。

---

## RC-MING-GANG-002 报听后允许明杠

Title：

报听后允许明杠。

Related Rules：

- GR-034

Action：

MING_GANG

Precondition：

玩家已报听。

明杠后仍保持听牌。

Expected：

允许明杠。

允许继续摸杠尾牌。

Broadcast：

```text
MING_GANG_SUCCESS
```

Notes：

必须再次验证：

仍然听牌。

---

## RC-MING-GANG-003 报听后禁止失听明杠

Title：

报听后明杠导致失听。

Related Rules：

- GR-034

Action：

MING_GANG

Expected：

服务器拒绝明杠。

Broadcast：

```text
MING_GANG_FORBIDDEN
```

Notes：

任何杠牌不得破坏听牌。

---

## RC-MING-GANG-004 明杠后允许杠开

Title：

明杠后摸牌胡牌。

Related Rules：

- GR-049

Action：

MING_GANG

Expected：

若杠尾摸牌胡牌。

Event：

```text
GANG_KAI
```

成立。

Flower Calculation：

```text
杠开 +2
```

---

# 九、BU_GANG 测试案例

## RC-BU-GANG-001 正常补杠

Title：

碰牌后补杠。

Related Rules：

- GR-023

Action：

BU_GANG

Precondition：

已经碰：

```text
5条
```

之后摸到：

```text
5条
```

Expected：

碰牌升级为补杠。

Broadcast：

```text
BU_GANG_SUCCESS

DRAW_FROM_GANG
```

Flower Calculation：

```text
补杠 +2
```

---

## RC-BU-GANG-002 补杠后必须出牌

Title：

补杠完成继续游戏。

Related Rules：

- GR-023

Action：

BU_GANG

Expected：

杠尾摸牌。

然后：

玩家必须继续打一张牌。

Notes：

补杠不是直接结束回合。

---

## RC-BU-GANG-003 报听后允许补杠

Title：

报听后补杠。

Related Rules：

- GR-034

Action：

BU_GANG

Expected：

若仍保持听牌。

允许补杠。

---

## RC-BU-GANG-004 报听后禁止失听补杠

Title：

补杠导致失听。

Expected：

服务器拒绝补杠。

Broadcast：

```text
BU_GANG_FORBIDDEN
```

---

## RC-BU-GANG-005 不存在抢杠胡

Title：

补杠过程中无人抢杠。

Related Rules：

- GR-023

Action：

BU_GANG

Expected：

服务器不会进入：

```text
QIANG_GANG_HU
```

流程。

Notes：

本麻将只有自摸。

不存在抢杠胡。

---

# 十、AN_GANG 测试案例

## RC-AN-GANG-001 正常暗杠

Title：

玩家正常暗杠。

Related Rules：

- GR-024

Action：

AN_GANG

Precondition：

玩家拥有：

```text
6筒
6筒
6筒
6筒
```

Expected：

暗杠成功。

Flower Calculation：

```text
暗杠 +4
```

Broadcast：

```text
AN_GANG_SUCCESS

DRAW_FROM_GANG
```

---

## RC-AN-GANG-002 玩家可放弃暗杠

Title：

暗杠不是强制操作。

Related Rules：

- GR-024

Action：

AN_GANG

Expected：

玩家可以：

选择暗杠。

也可以：

继续游戏。

Broadcast：

```text
WAIT_PLAYER_DECISION
```

Notes：

暗杠永远不是必须操作。

---

## RC-AN-GANG-003 报听后允许暗杠

Title：

报听后暗杠。

Related Rules：

- GR-034

Action：

AN_GANG

Expected：

若仍保持听牌。

允许暗杠。

---

## RC-AN-GANG-004 报听后禁止失听暗杠

Title：

暗杠导致失听。

Expected：

服务器拒绝暗杠。

Broadcast：

```text
AN_GANG_FORBIDDEN
```

---

## RC-AN-GANG-005 暗杠后杠开

Title：

暗杠后摸牌胡牌。

Related Rules：

- GR-049

Action：

AN_GANG

Expected：

Event：

```text
GANG_KAI
```

Flower Calculation：

```text
杠开 +2
```

---

## RC-AN-GANG-006 暗杠形成七对特殊判断

Title：

暗杠作为两对参与七对计算。

Related Rules：

- SP-011

Action：

WIN

Precondition：

玩家胡七对。

手牌存在暗杠。

Expected：

七对正常成立。

Flower Calculation：

```text
七对

+

暗杠组成两对

+

额外10张花
```

Notes：

这是你定义的特殊规则。

属于本麻将特色玩法。

---
