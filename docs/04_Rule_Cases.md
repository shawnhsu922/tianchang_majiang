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
