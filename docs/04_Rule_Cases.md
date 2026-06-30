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

# 二、测试原则

所有 Rule Case 必须满足：

1. 输入明确。
2. 输出明确。
3. 不使用“其他牌组”“类似情况”等模糊描述。
4. 所有牌型必须可以被程序复现。
5. 所有最终花数必须对应 `03_Scoring_Table.md`。

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

每个测试案例统一格式如下：

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
```

摸牌：

```text
1筒
```

Expected：

- 东家手牌增加 1 筒。
- 服务器进入等待东家出牌状态。
- 不触发胡牌。
- 不触发补花。
- 不触发杠牌。

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
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

基本花 +1，暂不结算。

Broadcast：

```text
DRAW_FLOWER
WAIT_BU_HUA
```

Notes：

东南西北中发白均属于基本花。

---

## RC-DRAW-003 摸牌后可胡牌

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
```

摸牌：

```text
9万
```

Expected：

- 服务器识别为可胡牌。
- 玩家可以选择胡牌。
- 玩家也可以选择放弃胡牌。
- 不得自动胡牌。

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

玩家摸到 9 万后组成四组面子加一对将。

---

## RC-DRAW-004 报听后摸到可胡牌但可放弃

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
```

摸牌：

```text
9万
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
2条 3条 4条
5条 6条 7条
6筒 7筒 8筒
9万
```

摸牌：

```text
3万
```

Expected：

- 服务器识别暗杠机会。
- 玩家可以选择暗杠。
- 玩家也可以选择不杠。
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

暗杠是可选操作。

---

## RC-DRAW-006 报听后摸牌形成暗杠且仍听

Related Rules：

- GR-026
- GR-035

Action：

DRAW

Precondition：

- 当前玩家已报听。
- 摸牌后形成暗杠。
- 暗杠后仍保持听牌。

Input：

当前玩家：

东家

报听：

是

当前手牌：

```text
3万 3万 3万
2条 3条 4条
5条 6条 7条
6筒 7筒 8筒
9万
```

摸牌：

```text
3万
```

Expected：

- 服务器识别暗杠机会。
- 服务器必须检查暗杠后是否仍然听牌。
- 若仍听，允许暗杠。

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

报听后可以杠，但杠后必须仍听。

---

## RC-DRAW-007 报听后摸牌形成暗杠但杠后不听

Related Rules：

- GR-026
- GR-035

Action：

DRAW

Precondition：

- 当前玩家已报听。
- 摸牌后形成暗杠。
- 暗杠后不再听牌。

Input：

当前玩家：

东家

报听：

是

当前手牌：

```text
3万 3万 3万
4万 5万
2条 3条 4条
5条 6条 7条
9筒 9筒
```

摸牌：

```text
3万
```

Expected：

- 服务器识别暗杠机会。
- 服务器检查暗杠后听牌状态。
- 若暗杠后失去听牌，禁止暗杠。

Score：

N/A

Broadcast：

```text
AN_GANG_FORBIDDEN_NOT_TING
```

Notes：

报听后禁止导致失听的杠牌。

---

## RC-DRAW-008 摸牌后形成补杠机会

Related Rules：

- GR-023
- GR-025

Action：

DRAW

Precondition：

- 当前玩家之前已经碰过 5 条。
- 当前摸到第 4 张 5 条。

Input：

当前玩家：

东家

碰牌：

```text
5条 5条 5条
```

当前手牌：

```text
2万 3万 4万
3万 4万 5万
6筒 7筒 8筒
9万 9万
```

摸牌：

```text
5条
```

Expected：

- 服务器识别补杠机会。
- 玩家可以选择补杠。
- 玩家也可以选择不补杠。
- 不存在抢杠胡。

Score：

若补杠：

```text
补杠 +2
```

Broadcast：

```text
CAN_BU_GANG
WAIT_PLAYER_DECISION
```

Notes：

碰牌后摸到第四张，可以补杠。

---

## RC-DRAW-009 摸到最后一张并胡牌

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
```

摸牌：

```text
9万
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

## RC-DRAW-010 摸到最后一张但不胡牌则流局

Related Rules：

- GR-006
- GR-050

Action：

DRAW

Precondition：

- 当前牌墙只剩最后一张有效牌。
- 当前玩家摸到后不能胡牌。

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
2万 3万 4万
5万 6万 7万
3条 4条 5条
8筒
```

摸牌：

```text
9筒
```

Expected：

- 玩家不能胡牌。
- 若牌墙已空，进入流局。
- 流局后换庄。

Score：

N/A

Broadcast：

```text
DRAW_SUCCESS
ROUND_DRAW
CHANGE_DEALER
```

Notes：

最后一张未胡，直接流局。

---

## RC-DRAW-011 杠后摸牌胡牌

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
```

摸牌：

```text
9万
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

## RC-DRAW-012 海底捞月与杠开同时成立

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
2万 3万 4万
3万 4万 5万
4条 5条 6条
6筒 7筒 8筒
9万
```

摸牌：

```text
9万
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

Notes：

这是回归测试案例，防止海底与杠开被错误设为互斥。

---
