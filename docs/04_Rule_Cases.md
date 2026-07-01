# 天长麻将微信小程序（Tianchang Mahjong Mini Program）

# 04_Rule_Cases

Version：1.0.0

Status：Draft

Author：Shawn Hsu & ChatGPT

---

# 一、文档目的

本文档用于定义天长麻将服务器规则测试规范（Rule Test Specification）。

所有测试案例均用于验证：

- 游戏流程
- 规则判定
- 状态迁移
- 胡牌判定
- 花数计算
- 特殊规则
- 回归测试

本文件是麻将服务器开发阶段唯一测试依据。

---

# 二、测试原则

所有 Rule Case 必须遵循以下原则：

1、所有输入均为构造测试数据，不要求来源于真实牌局。

2、测试数据必须符合麻将牌数量限制。

3、同一种牌最多出现四张。

4、每一个 Rule Case 必须只有唯一预期结果。

5、每一个 Rule Case 必须能够转换为自动化测试。

6、所有花数均以《03_Scoring_Table.md》为唯一标准。

---

# 三、服务器动作（Server Actions）

麻将服务器仅允许处理以下动作：

| Action | 说明 |
|---------|------|
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

所有 Rule Case 均按照 Action 分类。

---

# 四、Rule Case 模板

所有测试案例统一采用以下格式。

## RC-XXX-001

Priority：

P0

Title：

测试名称

Related Rules：

关联规则编号。

Action：

服务器动作。

Purpose：

本测试目的。

Precondition：

测试前置条件。

Input：

测试输入。

Expected：

服务器预期结果。

Broadcast：

服务器广播。

Notes：

补充说明。

---

# 五、DRAW（摸牌）

DRAW 为整个麻将服务器最核心动作。

服务器收到 DRAW 后，必须依次执行：

1、玩家摸牌。

2、检查是否摸到基本花。

3、若摸到基本花，进入补花流程。

4、若不是基本花，检查是否胡牌。

5、检查是否形成暗杠。

6、检查是否形成补杠。

7、若均不成立，进入等待出牌状态。

所有测试案例均围绕上述流程展开。

---

## RC-DRAW-001 普通摸牌

Priority：

P0

Title：

普通摸牌进入等待出牌状态。

Related Rules：

- GR-013

Action：

DRAW

Purpose：

验证玩家摸到普通牌后，服务器能够正确进入等待出牌状态。

Precondition：

- 游戏已经开始。
- 当前轮到东家摸牌。
- 东家未报听。
- 摸到的牌不是基本花。
- 摸牌后不能胡牌。
- 摸牌后没有杠牌机会。

Input：

```text
Current State：
WAIT_DRAW

Draw Card：
9筒
```

Expected：

```text
Next State：
WAIT_DISCARD
```

服务器应完成：

- 手牌增加一张。
- 当前玩家获得出牌权。
- 不触发胡牌检查结果。
- 不触发杠牌操作。
- 不进入补花流程。

Broadcast：

```text
DRAW_SUCCESS

WAIT_DISCARD
```

Notes：

标准摸牌流程。

---

## RC-DRAW-002 摸到基本花

Priority：

P0

Title：

摸到基本花进入补花流程。

Related Rules：

- GR-010
- GR-013

Action：

DRAW

Purpose：

验证服务器收到基本花后立即进入补花流程。

Precondition：

游戏已经开始。

Input：

```text
Current State：
WAIT_DRAW

Draw Card：
东风
```

Expected：

```text
Next State：
WAIT_BU_HUA
```

服务器应完成：

- 基本花加入花牌区。
- 当前玩家不能立即出牌。
- 从杠尾补牌。

Broadcast：

```text
DRAW_FLOWER

WAIT_BU_HUA
```

Notes：

东南西北中发白均属于基本花。

---

## RC-DRAW-003 摸牌形成可胡状态

Priority：

P0

Title：

摸牌后服务器识别可以胡牌。

Related Rules：

- GR-039

Action：

DRAW

Purpose：

验证服务器正确识别自摸胡牌。

Precondition：

摸牌后形成合法胡牌。

Input：

```text
Current State：
WAIT_DRAW

Draw Card：
可胡牌
```

Expected：

```text
Next State：
WAIT_PLAYER_DECISION
```

服务器应完成：

- 标记可以胡牌。
- 等待玩家决定。
- 不自动胡牌。

Broadcast：

```text
CAN_WIN

WAIT_PLAYER_DECISION
```

Notes：

允许玩家选择胡或放弃。

---

## RC-DRAW-004 摸牌后不能胡牌

Priority：

P0

Title：

摸牌后不能胡牌继续游戏。

Related Rules：

- GR-039

Action：

DRAW

Purpose：

验证不能胡牌时正常进入出牌阶段。

Precondition：

摸牌后不能胡。

Input：

```text
Current State：
WAIT_DRAW
```

Expected：

```text
Next State：
WAIT_DISCARD
```

服务器应完成：

- 不提示胡牌。
- 玩家继续出牌。

Broadcast：

```text
WAIT_DISCARD
```

---

## RC-DRAW-005 摸牌形成暗杠

Priority：

P0

Title：

摸牌形成暗杠机会。

Related Rules：

- GR-024

Action：

DRAW

Purpose：

验证服务器能够识别暗杠机会。

Precondition：

摸牌后拥有四张相同牌。

Input：

```text
Current State：
WAIT_DRAW
```

Expected：

```text
Next State：
WAIT_PLAYER_DECISION
```

服务器应完成：

- 提示可以暗杠。
- 玩家可选择暗杠。
- 玩家可选择放弃。

Broadcast：

```text
CAN_AN_GANG
```

Notes：

暗杠不是强制操作。

---

## RC-DRAW-006 摸牌形成补杠

Priority：

P0

Title：

摸牌形成补杠机会。

Related Rules：

- GR-023

Action：

DRAW

Purpose：

验证服务器识别补杠机会。

Precondition：

玩家之前已经碰牌。

摸到第四张。

Expected：

```text
Next State：
WAIT_PLAYER_DECISION
```

服务器应完成：

- 提示可以补杠。
- 玩家可补杠。
- 玩家可放弃。

Broadcast：

```text
CAN_BU_GANG
```

---

## RC-DRAW-007 摸到最后一张胡牌

Priority：

P0

Title：

海底捞月。

Related Rules：

- GR-050

Action：

DRAW

Purpose：

验证最后一张有效牌胡牌。

Precondition：

当前牌墙剩余最后一张有效牌。

Expected：

Event：

```text
HAI_DI
```

Flower：

```text
海底捞月 +10
```

Broadcast：

```text
EVENT_HAI_DI
```

---

## RC-DRAW-008 杠尾最后一张胡牌

Priority：

P0

Title：

海底捞月与杠开同时成立。

Related Rules：

- GR-049
- GR-050
- GR-051

Action：

DRAW

Purpose：

验证最后一张有效牌来自杠尾。

Precondition：

玩家补花或杠牌。

摸到最后一张有效牌。

Expected：

Events：

```text
HAI_DI

GANG_KAI
```

Flower Calculation：

```text
海底捞月 +10

杠开 +2
```

Broadcast：

```text
EVENT_HAI_DI

EVENT_GANG_KAI
```

Notes：

两个事件允许同时成立。

---

## RC-DRAW-009 最后一张无人胡牌

Priority：

P0

Title：

流局。

Related Rules：

- GR-006

Action：

DRAW

Purpose：

验证最后一张摸完无人胡牌。

Expected：

```text
Next State：

SETTLEMENT
```

Broadcast：

```text
ROUND_DRAW
```

Notes：

本局结束。

---

## RC-DRAW-010 报听后摸牌

Priority：

P0

Title：

报听玩家正常摸牌。

Related Rules：

- GR-034

Action：

DRAW

Purpose：

验证报听玩家摸牌后仍按正常流程检查。

Expected：

服务器依次检查：

- 是否胡牌；
- 是否杠牌；
- 是否保持听牌；

最终进入：

WAIT_PLAYER_DECISION

或

WAIT_DISCARD。

---

# 六、DISCARD（出牌）

DISCARD 为服务器第二核心动作。

服务器收到 DISCARD 后，必须依次执行：

1、验证当前玩家是否拥有出牌权。

2、验证该牌是否属于当前玩家手牌。

3、移除手牌。

4、加入弃牌区。

5、按逆时针顺序检查：

- 碰
- 明杠

（本麻将没有吃。）

6、若无人响应。

进入下一家 DRAW。

---

## RC-DISCARD-001 普通出牌

Priority：

P0

Title：

普通出牌。

Related Rules：

- GR-014

Action：

DISCARD

Purpose：

验证正常出牌流程。

Precondition：

当前玩家拥有出牌权。

Expected：

Next State：

```text
WAIT_RESPONSE
```

服务器应完成：

- 手牌减少一张。
- 弃牌区增加一张。
- 广播所有玩家。

Broadcast：

```text
DISCARD_SUCCESS

WAIT_RESPONSE
```

---

## RC-DISCARD-002 无出牌权禁止出牌

Priority：

P0

Title：

没有出牌权。

Related Rules：

- GR-014

Action：

DISCARD

Purpose：

验证非法出牌。

Precondition：

当前玩家不是当前行动玩家。

Expected：

服务器拒绝操作。

Broadcast：

```text
DISCARD_FORBIDDEN
```

Notes：

服务器不得修改任何状态。

---

## RC-DISCARD-003 打出不存在的牌

Priority：

P0

Title：

非法牌。

Related Rules：

- GR-014

Action：

DISCARD

Purpose：

验证非法数据。

Input：

玩家打出的牌。

不存在于自己手牌。

Expected：

服务器拒绝。

Broadcast：

```text
INVALID_CARD
```

---

## RC-DISCARD-004 无人响应

Priority：

P0

Title：

无人碰杠。

Related Rules：

- GR-014

Action：

DISCARD

Purpose：

验证无人响应流程。

Expected：

服务器进入：

```text
NEXT_PLAYER_DRAW
```

Broadcast：

```text
NO_RESPONSE

NEXT_PLAYER_DRAW
```

---

## RC-DISCARD-005 有玩家可以碰

Priority：

P0

Title：

发现碰牌机会。

Related Rules：

- GR-015

Action：

DISCARD

Purpose：

验证服务器识别碰牌。

Expected：

服务器：

等待碰牌玩家。

Broadcast：

```text
CAN_PENG

WAIT_PLAYER_DECISION
```

---

## RC-DISCARD-006 有玩家可以明杠

Priority：

P0

Title：

发现明杠机会。

Related Rules：

- GR-022

Action：

DISCARD

Purpose：

验证服务器识别明杠。

Expected：

服务器：

等待玩家决定。

Broadcast：

```text
CAN_MING_GANG

WAIT_PLAYER_DECISION
```

---

## RC-DISCARD-007 报听玩家出牌

Priority：

P0

Title：

报听后正常出牌。

Related Rules：

- GR-034

Action：

DISCARD

Purpose：

验证报听玩家正常打牌。

Expected：

服务器：

允许出牌。

保持：

TING 状态。

Broadcast：

```text
DISCARD_SUCCESS
```

---

## RC-DISCARD-008 报听玩家非法换牌

Priority：

P0

Title：

报听后改变听口。

Related Rules：

- GR-034

Action：

DISCARD

Purpose：

验证服务器禁止失听。

Expected：

服务器拒绝。

Broadcast：

```text
NOT_TING_ANYMORE
```

Notes：

报听后不得改变听牌状态。

---

# 七、PENG（碰牌）

服务器收到：

PENG

必须验证：

1、是否轮到碰牌。

2、是否拥有两张相同牌。

3、是否允许碰。

4、碰后立即获得出牌权。

---

## RC-PENG-001 正常碰牌

Priority：

P0

Title：

正常碰牌。

Related Rules：

- GR-015

Action：

PENG

Purpose：

验证正常碰牌流程。

Expected：

Next State：

```text
WAIT_DISCARD
```

服务器：

- 建立碰牌组。
- 玩家获得出牌权。

Broadcast：

```text
PENG_SUCCESS

WAIT_DISCARD
```

---

## RC-PENG-002 碰牌失败

Priority：

P0

Title：

没有两张相同牌。

Related Rules：

- GR-015

Action：

PENG

Purpose：

验证非法碰牌。

Expected：

服务器拒绝。

Broadcast：

```text
PENG_FORBIDDEN
```

---

## RC-PENG-003 报听禁止碰

Priority：

P0

Title：

报听后不能碰。

Related Rules：

- GR-034

Action：

PENG

Purpose：

验证报听限制。

Expected：

服务器拒绝。

Broadcast：

```text
PENG_FORBIDDEN_AFTER_TING
```

---

## RC-PENG-004 碰牌后必须出牌

Priority：

P0

Title：

碰牌继续流程。

Related Rules：

- GR-015

Action：

PENG

Purpose：

验证碰牌后继续游戏。

Expected：

服务器：

等待出牌。

Broadcast：

```text
WAIT_DISCARD
```

---

## RC-PENG-005 碰牌组锁定

Priority：

P0

Title：

碰牌后不能拆牌。

Related Rules：

- GR-029

Action：

PENG

Purpose：

验证公开牌锁定。

Expected：

服务器：

锁定碰牌组。

不得拆分。

Broadcast：

```text
LOCK_PENG_GROUP
```

Notes：

碰牌组以后只能：

补杠。

不得再次组成顺子或回收进入手牌。

---

---

# 八、MING_GANG（明杠）

服务器收到：

MING_GANG

必须验证：

1、当前玩家是否拥有三张相同牌。

2、该牌是否来自其他玩家刚打出的牌。

3、当前玩家是否允许明杠。

4、明杠成功后进入杠尾摸牌。

---

## RC-MING-GANG-001 正常明杠

Priority：

P0

Title：

正常明杠。

Related Rules：

- GR-022

Action：

MING_GANG

Purpose：

验证服务器正常执行明杠。

Precondition：

玩家拥有三张相同牌。

Expected：

Next State：

```text
WAIT_GANG_DRAW
```

服务器应完成：

- 建立公开杠牌组。
- 增加明杠花数。
- 进入杠尾摸牌。

Broadcast：

```text
MING_GANG_SUCCESS

WAIT_GANG_DRAW
```

---

## RC-MING-GANG-002 报听后允许明杠

Priority：

P0

Title：

报听后仍保持听牌允许明杠。

Related Rules：

- GR-034

Action：

MING_GANG

Purpose：

验证报听后合法明杠。

Expected：

服务器验证：

```text
Gang After Ting
```

仍保持听牌。

允许明杠。

Broadcast：

```text
MING_GANG_SUCCESS
```

---

## RC-MING-GANG-003 报听后失听禁止明杠

Priority：

P0

Title：

明杠导致失听。

Related Rules：

- GR-034

Action：

MING_GANG

Purpose：

验证服务器禁止失听。

Expected：

服务器拒绝。

Broadcast：

```text
MING_GANG_FORBIDDEN
```

---

## RC-MING-GANG-004 明杠后胡牌

Priority：

P0

Title：

杠开。

Related Rules：

- GR-049

Action：

MING_GANG

Purpose：

验证杠尾摸牌胡牌。

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

# 九、BU_GANG（补杠）

服务器收到：

BU_GANG

必须验证：

1、之前已经碰牌。

2、摸到第四张。

3、升级公开牌组。

4、继续杠尾摸牌。

---

## RC-BU-GANG-001 正常补杠

Priority：

P0

Title：

碰牌升级补杠。

Related Rules：

- GR-023

Action：

BU_GANG

Purpose：

验证补杠。

Expected：

服务器：

升级公开牌组。

Broadcast：

```text
BU_GANG_SUCCESS
```

---

## RC-BU-GANG-002 补杠后继续摸牌

Priority：

P0

Title：

补杠继续游戏。

Related Rules：

- GR-023

Action：

BU_GANG

Purpose：

验证杠尾摸牌。

Expected：

Next State：

```text
WAIT_GANG_DRAW
```

Broadcast：

```text
WAIT_GANG_DRAW
```

---

## RC-BU-GANG-003 报听允许补杠

Priority：

P0

Title：

补杠后保持听牌。

Related Rules：

- GR-034

Action：

BU_GANG

Expected：

允许。

Broadcast：

```text
BU_GANG_SUCCESS
```

---

## RC-BU-GANG-004 补杠导致失听

Priority：

P0

Title：

禁止补杠。

Related Rules：

- GR-034

Action：

BU_GANG

Expected：

服务器拒绝。

Broadcast：

```text
BU_GANG_FORBIDDEN
```

---

## RC-BU-GANG-005 无抢杠胡

Priority：

P0

Title：

补杠流程。

Related Rules：

- GR-023

Action：

BU_GANG

Purpose：

验证本麻将没有抢杠胡。

Expected：

服务器：

不会进入：

```text
QIANG_GANG_HU
```

Broadcast：

```text
BU_GANG_SUCCESS
```

Notes：

本麻将只有自摸。

---

# 十、AN_GANG（暗杠）

服务器收到：

AN_GANG

必须验证：

1、玩家拥有四张相同牌。

2、允许暗杠。

3、进入杠尾摸牌。

---

## RC-AN-GANG-001 正常暗杠

Priority：

P0

Title：

正常暗杠。

Related Rules：

- GR-024

Action：

AN_GANG

Purpose：

验证暗杠流程。

Expected：

服务器：

建立暗杠牌组。

Broadcast：

```text
AN_GANG_SUCCESS

WAIT_GANG_DRAW
```

Flower Calculation：

```text
暗杠 +4
```

---

## RC-AN-GANG-002 玩家放弃暗杠

Priority：

P0

Title：

暗杠不是必须操作。

Related Rules：

- GR-024

Action：

AN_GANG

Expected：

允许：

PASS。

Broadcast：

```text
WAIT_PLAYER_DECISION
```

---

## RC-AN-GANG-003 报听允许暗杠

Priority：

P0

Title：

暗杠后保持听牌。

Related Rules：

- GR-034

Action：

AN_GANG

Expected：

允许暗杠。

Broadcast：

```text
AN_GANG_SUCCESS
```

---

## RC-AN-GANG-004 暗杠导致失听

Priority：

P0

Title：

禁止暗杠。

Related Rules：

- GR-034

Action：

AN_GANG

Expected：

服务器拒绝。

Broadcast：

```text
AN_GANG_FORBIDDEN
```

---

## RC-AN-GANG-005 暗杠后杠开

Priority：

P0

Title：

暗杠后胡牌。

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

## RC-AN-GANG-006 七对暗杠特殊规则

Priority：

P1

Title：

暗杠作为两对参与七对计算。

Related Rules：

- SP-011

Action：

WIN

Purpose：

验证特色规则。

Expected：

Flower Calculation：

```text
七对

+

暗杠组成两对

+

额外10张
```

Notes：

属于天长麻将特色规则。

---

---

---

# 十一、BU_HUA（补花）

服务器收到：

BU_HUA

必须验证：

1、玩家摸到基本花。

2、基本花加入花牌区。

3、从杠尾补牌。

4、若再次摸到基本花，则继续补花。

5、若补到普通牌，则结束补花流程。

6、补花结束后继续正常游戏流程。

---

## RC-BU-HUA-001 正常补花

Priority：

P0

Title：

摸到基本花后正常补花。

Related Rules：

- GR-010

Action：

BU_HUA

Purpose：

验证补花基本流程。

Precondition：

玩家摸到基本花。

Input：

```text
Current State：
WAIT_BU_HUA

Flower Card：
东风
```

Expected：

Next State：

```text
WAIT_GANG_DRAW
```

服务器应完成：

- 将东风加入玩家花牌区。
- 增加基本花数量。
- 从杠尾补一张牌。
- 玩家不能立即出牌。

Broadcast：

```text
BU_HUA_SUCCESS

DRAW_FROM_GANG
```

Notes：

东、南、西、北、中、发、白均属于基本花。

---

## RC-BU-HUA-002 连续补花

Priority：

P0

Title：

补花后再次补到基本花。

Related Rules：

- GR-010

Action：

BU_HUA

Purpose：

验证连续补花流程。

Precondition：

第一次补花完成。

杠尾再次补到基本花。

Input：

```text
First Flower：
东风

Second Flower：
白板
```

Expected：

Next State：

```text
WAIT_BU_HUA
```

服务器应完成：

- 东风加入花牌区。
- 白板加入花牌区。
- 再次进入补花流程。
- 不允许玩家进行任何其他操作。

Broadcast：

```text
BU_HUA_SUCCESS

WAIT_BU_HUA
```

Notes：

连续补到基本花时，必须一直补花，直到补到普通牌。

---

## RC-BU-HUA-003 补花结束

Priority：

P0

Title：

补花后补到普通牌。

Related Rules：

- GR-010

Action：

BU_HUA

Purpose：

验证补花结束流程。

Precondition：

玩家完成补花。

杠尾补到普通牌。

Input：

```text
Flower：
东风

Gang Draw：
五万
```

Expected：

服务器应完成：

- 东风加入花牌区。
- 五万加入玩家手牌。
- 补花流程结束。

随后立即进入正常摸牌判定流程。

服务器继续检查：

- 是否可胡牌。
- 是否可暗杠。
- 是否可补杠。
- 是否需要玩家出牌。

Broadcast：

```text
BU_HUA_FINISHED

CHECK_PLAYER_ACTION
```

Notes：

补到普通牌后，不允许直接进入出牌阶段。

必须重新执行完整的玩家操作检查。

---

## RC-BU-HUA-004 补花结束立即天胡

Priority：

P0

Title：

补花结束立即胡牌。

Related Rules：

- GR-010
- GR-046

Action：

BU_HUA

Purpose：

验证补花结束立即胡牌判定为天胡。

Precondition：

当前仍属于开局补花阶段。

玩家补花结束。

形成合法胡牌。

Input：

```text
Current Phase：
INITIAL_BU_HUA

After BU_HUA：
Can Win
```

Expected：

服务器应识别：

```text
Can Win：
true

Event：
TIAN_HU
```

Flower Calculation：

```text
天胡

200张花
```

不得识别：

```text
GANG_KAI

HAI_DI_LAO_YUE
```

Broadcast：

```text
CAN_WIN

EVENT_TIAN_HU

WAIT_PLAYER_DECISION
```

Notes：

补花结束立即胡牌属于天胡。

天胡与杠开、海底捞月互斥。

---

## RC-BU-HUA-005 庄家无花天胡

Priority：

P0

Title：

庄家无基本花直接天胡。

Related Rules：

- GR-046

Action：

INITIAL_DRAW

Purpose：

验证庄家没有基本花时第一次摸牌即可天胡。

Precondition：

庄家没有基本花。

无需补花。

第一张摸牌即可胡牌。

Input：

```text
Player：
Dealer

Need BU_HUA：
false

Can Win：
true
```

Expected：

服务器识别：

```text
Event：
TIAN_HU
```

Flower Calculation：

```text
天胡

200张花
```

Broadcast：

```text
CAN_WIN

EVENT_TIAN_HU
```

Notes：

庄家没有基本花，不需要经过补花流程。

第一次摸牌即可判定天胡。

---

## RC-BU-HUA-006 闲家补花天胡

Priority：

P0

Title：

闲家补花结束立即胡牌。

Related Rules：

- GR-046

Action：

BU_HUA

Purpose：

验证闲家补花结束立即胡牌。

Precondition：

庄家补花结束。

庄家未胡牌。

轮到闲家补花。

闲家补花结束立即形成合法胡牌。

Input：

```text
Player：
South

Current Phase：
INITIAL_BU_HUA

Can Win：
true
```

Expected：

服务器识别：

```text
Event：
TIAN_HU
```

Flower Calculation：

```text
天胡

200张花
```

Broadcast：

```text
CAN_WIN

EVENT_TIAN_HU
```

Notes：

按照天长麻将规则。

补花顺序中任何玩家补花结束立即胡牌。

均属于天胡。

---

## RC-BU-HUA-007 连续补花后天胡

Priority：

P1

Title：

连续补花结束后立即天胡。

Related Rules：

- GR-010
- GR-046

Action：

BU_HUA

Purpose：

验证连续补花不会影响天胡判定。

Precondition：

玩家连续补花。

直到最后一次补到普通牌。

立即形成合法胡牌。

Input：

```text
Flowers：

东风

白板

红中

Final Draw：

Can Win
```

Expected：

服务器识别：

```text
Event：
TIAN_HU
```

Flower Calculation：

```text
天胡

200张花
```

Broadcast：

```text
CAN_WIN

EVENT_TIAN_HU
```

Notes：

只要仍属于开局补花阶段。

无论连续补花多少次。

最终补花结束立即胡牌。

均属于天胡。

---

---

# 十二、TING（报听）

服务器收到：

TING

必须验证：

1、玩家主动选择报听。

2、当前手牌必须已经听牌。

3、服务器计算所有可胡牌。

4、进入报听状态。

5、广播玩家已报听。

6、后续禁止主动改变手牌结构。

---

## RC-TING-001 正常报听

Priority：

P0

Title：

玩家正常报听。

Related Rules：

- GR-031

Action：

TING

Purpose：

验证玩家正常进入报听状态。

Precondition：

玩家已经听牌。

主动点击：

报听。

Input：

```text
Can Ting：
true

Player Action：
TING
```

Expected：

服务器验证：

```text
Can Ting：
true
```

进入状态：

```text
PLAYER_TING
```

服务器保存：

- 报听状态
- 当前听牌列表

Broadcast：

```text
PLAYER_TING

WAIT_DRAW
```

Notes：

报听必须由玩家主动操作。

---

## RC-TING-002 未听牌禁止报听

Priority：

P0

Title：

未满足听牌条件。

Related Rules：

- GR-031

Action：

TING

Purpose：

验证服务器拒绝非法报听。

Precondition：

玩家尚未听牌。

Input：

```text
Can Ting：
false

Player Action：
TING
```

Expected：

服务器拒绝。

保持原状态。

Broadcast：

```text
TING_FAILED
```

Notes：

程序不得允许假报听。

---

## RC-TING-003 报听后禁止主动换牌

Priority：

P0

Title：

报听后禁止改变手牌结构。

Related Rules：

- GR-032

Action：

DISCARD

Purpose：

验证报听限制。

Precondition：

玩家已经报听。

Expected：

服务器禁止：

- 换搭子。
- 拆对子。
- 改变听牌结构。

服务器只允许：

当前摸牌。

↓

胡牌。

或：

打出刚摸到的牌。

Broadcast：

```text
TING_LOCK
```

Notes：

报听后手牌必须保持听牌状态。

不得主动改变牌型。

---

## RC-TING-004 报听后允许暗杠

Priority：

P0

Title：

报听后允许暗杠。

Related Rules：

- GR-024
- GR-032

Action：

AN_GANG

Purpose：

验证报听后允许暗杠。

Precondition：

玩家已经报听。

当前拥有合法暗杠。

暗杠后仍保持听牌。

Input：

```text
Player：
TING

Can AN_GANG：
true

After Gang：
Still Ting
```

Expected：

服务器验证：

```text
Still Ting：
true
```

允许暗杠。

进入杠尾摸牌流程。

Broadcast：

```text
AN_GANG_SUCCESS

WAIT_GANG_DRAW
```

Notes：

报听后允许暗杠。

前提是暗杠后不能失去听牌状态。

---

## RC-TING-005 报听后允许明杠

Priority：

P0

Title：

报听后允许明杠。

Related Rules：

- GR-022
- GR-032

Action：

MING_GANG

Purpose：

验证报听后允许明杠。

Precondition：

玩家已经报听。

满足明杠条件。

明杠后仍保持听牌。

Input：

```text
Player：
TING

Can MING_GANG：
true

After Gang：
Still Ting
```

Expected：

服务器验证：

```text
Still Ting：
true
```

允许明杠。

Broadcast：

```text
MING_GANG_SUCCESS

WAIT_GANG_DRAW
```

Notes：

报听后允许明杠。

不得破坏听牌状态。

---

## RC-TING-006 报听后允许补杠

Priority：

P0

Title：

报听后允许补杠。

Related Rules：

- GR-023
- GR-032

Action：

BU_GANG

Purpose：

验证报听后允许补杠。

Precondition：

玩家已经报听。

拥有碰牌。

摸到第四张。

补杠后仍保持听牌。

Input：

```text
Player：
TING

Can BU_GANG：
true

After Gang：
Still Ting
```

Expected：

服务器验证：

```text
Still Ting：
true
```

允许补杠。

Broadcast：

```text
BU_GANG_SUCCESS

WAIT_GANG_DRAW
```

Notes：

报听后允许补杠。

不得导致失听。

---

## RC-TING-007 杠后仍保持听牌

Priority：

P0

Title：

报听后杠牌仍保持听牌。

Related Rules：

- GR-032
- GR-035

Action：

GANG_CHECK

Purpose：

验证报听玩家杠牌后仍保持听牌时，服务器允许杠牌。

Precondition：

玩家已经报听。

玩家发起杠牌操作。

杠牌后仍然存在可胡牌。

Input：

```text
Player：
TING

Before Gang：
Can Win Tiles Exist

After Gang：
Can Win Tiles Exist
```

Expected：

服务器验证：

```text
Still Ting：
true
```

允许杠牌。

Broadcast：

```text
GANG_ALLOWED

WAIT_GANG_DRAW
```

Notes：

报听后可以杠，但必须保持听牌。

---

## RC-TING-008 杠后失听禁止杠

Priority：

P0

Title：

报听后杠牌导致失听。

Related Rules：

- GR-032
- GR-035

Action：

GANG_CHECK

Purpose：

验证报听玩家杠牌后失去听牌时，服务器禁止杠牌。

Precondition：

玩家已经报听。

玩家发起杠牌操作。

杠牌后不再存在可胡牌。

Input：

```text
Player：
TING

Before Gang：
Can Win Tiles Exist

After Gang：
No Win Tiles
```

Expected：

服务器验证：

```text
Still Ting：
false
```

拒绝杠牌。

保持原手牌状态。

Broadcast：

```text
GANG_FORBIDDEN_NOT_TING
```

Notes：

报听后禁止任何导致失听的杠牌。

---

## RC-TING-009 报听后可胡但允许放弃

Priority：

P0

Title：

报听后摸到胡牌可以选择不胡。

Related Rules：

- GR-036

Action：

WIN_DECISION

Purpose：

验证报听玩家摸到可胡牌时，服务器不得强制胡牌。

Precondition：

玩家已经报听。

玩家摸到可胡牌。

Input：

```text
Player：
TING

Can Win：
true

Player Decision：
PASS
```

Expected：

服务器允许玩家放弃胡牌。

游戏继续。

若玩家放弃胡牌：

```text
Win：
false
```

Broadcast：

```text
WIN_AVAILABLE

PLAYER_PASS_WIN
```

Notes：

即使已经报听，摸到胡牌也可以不胡。

---

## RC-TING-010 重连后恢复报听状态

Priority：

P1

Title：

玩家重连后恢复报听状态。

Related Rules：

- GR-032

Action：

RECONNECT

Purpose：

验证玩家重连后，服务器正确恢复报听状态。

Precondition：

玩家已经报听。

游戏过程中网络断开。

随后重新连接。

Input：

```text
Player：
TING

Reconnect：
true
```

Expected：

服务器恢复：

- 报听状态。
- 当前手牌。
- 当前听牌列表。
- 当前轮次。

Broadcast：

```text
PLAYER_RECONNECTED

PLAYER_TING
```

Notes：

重连不得丢失报听状态。

---

## RC-TING-011 托管状态保持报听

Priority：

P1

Title：

玩家进入托管后保持报听状态。

Related Rules：

- GR-032

Action：

TRUSTEE

Purpose：

验证托管不会取消报听状态。

Precondition：

玩家已经报听。

玩家进入托管。

Input：

```text
Player：
TING

Trustee：
true
```

Expected：

服务器保持：

```text
Player State：
TING
```

托管玩家：

仍然按照报听规则进行游戏。

Broadcast：

```text
PLAYER_TRUSTEE
```

Notes：

托管不会取消报听。

---

## RC-TING-012 广播报听状态

Priority：

P1

Title：

玩家报听后广播状态。

Related Rules：

- GR-031

Action：

TING

Purpose：

验证服务器正确广播报听。

Precondition：

玩家成功报听。

Input：

```text
Player：
TING
```

Expected：

服务器广播：

```text
PLAYER_TING
```

其他三家客户端应看到：

- 玩家已报听。
- 玩家昵称旁显示报听标识。
- 后续游戏按照报听规则进行。

Broadcast：

```text
PLAYER_TING
```

Notes：

广播仅同步状态。

不得泄露玩家听哪些牌。

---

---

# 十三、WIN（胡牌）

WIN 为整个麻将服务器最核心动作。

服务器收到：

WIN

必须按照固定顺序完成判定。

判定顺序如下：

1、验证是否为合法胡牌。

2、识别基础胡型。

3、识别特殊牌型。

4、识别胡牌事件。

5、计算最终花数。

6、生成结算数据。

任何一步失败。

立即停止后续判定。

---

## RC-WIN-001 非法胡牌

Priority：

P0

Title：

不能组成合法胡牌。

Related Rules：

- GR-039

Action：

WIN

Purpose：

验证非法胡牌。

Precondition：

玩家点击胡牌。

Input：

```text
Can Win：
false
```

Expected：

服务器拒绝胡牌。

Broadcast：

```text
WIN_FAILED
```

Notes：

非法胡牌立即结束。

不得进入牌型分析。

---

## RC-WIN-002 合法普通胡牌

Priority：

P0

Title：

普通合法胡牌。

Related Rules：

- GR-039

Action：

WIN

Purpose：

验证合法胡牌。

Precondition：

玩家已经满足胡牌条件。

Input：

```text
Can Win：
true
```

Expected：

服务器继续：

```text
CHECK_BASE_TYPE
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

这里只验证：

是否合法。

不验证任何牌型。

---

## RC-WIN-003 先判定基础胡型

Priority：

P0

Title：

基础胡型优先。

Related Rules：

- GR-039

Action：

WIN

Purpose：

验证基础胡型优先识别。

Expected：

服务器首先识别：

- 多张
- 丫子
- 对倒
- 单吊
- 拐头丫子

识别完成后。

继续特殊牌型判断。

Broadcast：

```text
CHECK_BASE_TYPE
```

Notes：

基础胡型必须先于特殊牌型计算。

---

## RC-WIN-004 多张胡牌

Priority：

P0

Title：

多张胡牌判定。

Related Rules：

- SP-001

Action：

WIN

Purpose：

验证多张胡牌识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

34567条
345万
789筒
22万

自摸：

2条
```

Expected：

服务器识别：

```text
Base Type：

多张
```

Flower Calculation：

```text
多张

3张花
```

不得识别：

```text
丫子

对倒

单吊
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

34567 条胡 2/5/8 条均属于多张胡。

---

## RC-WIN-005 丫子胡牌

Priority：

P0

Title：

丫子胡牌判定。

Related Rules：

- SP-002

Action：

WIN

Purpose：

验证丫子胡牌识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

35条
345万
678万
789筒
22万

自摸：

4条
```

Expected：

服务器识别：

```text
Base Type：

丫子
```

Flower Calculation：

```text
丫子

5张花
```

不得识别：

```text
多张

对倒

单吊
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

35 条只胡 4 条。

属于丫子。

---

## RC-WIN-006 对倒胡牌

Priority：

P0

Title：

对倒胡牌判定。

Related Rules：

- SP-003

Action：

WIN

Purpose：

验证对倒胡牌识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

3344条
345万
678万
789筒
22万

自摸：

3条
```

Expected：

服务器识别：

```text
Base Type：

对倒
```

Flower Calculation：

```text
对倒

6张花
```

不得识别：

```text
多张

丫子

单吊
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

3344 胡 3 或 4。

均属于对倒。

---

## RC-WIN-007 单吊胡牌

Priority：

P0

Title：

单吊胡牌判定。

Related Rules：

- SP-004

Action：

WIN

Purpose：

验证单吊胡牌识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

11万
345万
678万
789筒
456条

自摸：

1万
```

Expected：

服务器识别：

```text
Base Type：

单吊
```

Flower Calculation：

```text
单吊

7张花
```

不得识别：

```text
多张

丫子

对倒
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

只有一张牌能够组成将牌胡牌。

---

## RC-WIN-008 拐头丫子

Priority：

P0

Title：

拐头丫子判定。

Related Rules：

- SP-005

Action：

WIN

Purpose：

验证拐头丫子识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

3445条
345万
678万
789筒
22万

自摸：

4条
```

Expected：

服务器识别：

```text
Base Type：

拐头丫子
```

Flower Calculation：

```text
拐头丫子

15张花
```

不得识别：

```text
丫子

多张

对倒
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

3445 只能胡4条。

属于拐头丫子。

---

## RC-WIN-009 对倒对对胡

Priority：

P0

Title：

对倒对对胡判定。

Related Rules：

- SP-006

Action：

WIN

Purpose：

验证对倒对对胡识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111万

222条

333筒

44万

55条

自摸：

4万
```

Expected：

服务器识别：

```text
Base Type：

对倒对对胡
```

Flower Calculation：

```text
对倒对对胡

16张花
```

不得识别：

```text
普通对倒
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

胡牌后整体牌型必须为对对胡。

若摸到其他牌形成普通胡牌，则不得按对对胡计分。

---

## RC-WIN-010 单吊对对胡

Priority：

P0

Title：

单吊对对胡判定。

Related Rules：

- SP-007

Action：

WIN

Purpose：

验证单吊对对胡识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111万

222条

333筒

444万

5条

自摸：

5条

## RC-WIN-013 七对

Priority：

P0

Title：

七对牌型判定。

Related Rules：

- SP-009

Action：

WIN

Purpose：

验证普通七对识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

11万

22万

33条

44条

55筒

66筒

7万

自摸：

7万
```

Expected：

服务器识别：

```text
Pattern：

七对
```

Flower Calculation：

```text
七对

27张花
```

不得识别：

```text
豪华七对
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

七对由七组对子组成。

---

## RC-WIN-014 清一色七对

Priority：

P0

Title：

清一色七对判定。

Related Rules：

- SP-010

Action：

WIN

Purpose：

验证清一色七对识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

11条

22条

33条

44条

55条

66条

7条

自摸：

7条
```

Expected：

服务器识别：

```text
Pattern：

七对

清一色
```

Flower Calculation：

```text
七对

27张花

+

清一色

30张花

=

57张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

清一色属于叠加牌型。

---

## RC-WIN-015 豪华七对

Priority：

P0

Title：

豪华七对判定。

Related Rules：

- SP-011

Action：

WIN

Purpose：

验证豪华七对识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111

33

44

66

77

99

自摸：

1
```

Expected：

服务器识别：

```text
Pattern：

豪华七对
```

Flower Calculation：

```text
豪华七对

67张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

必须满足：

手牌存在一个暗刻。

且只能胡该暗刻组成七对。

若还能胡其他牌，则不得判定豪华七对。

---

## RC-WIN-016 豪华七对（非豪华验证）

Priority：

P0

Title：

存在暗刻但不能判定豪华七对。

Related Rules：

- SP-011

Action：

WIN

Purpose：

验证豪华七对必须满足"只能胡暗刻"。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111万

22万

33条

44条

55筒

66筒

7万

自摸：

5筒
```

Expected：

服务器识别：

```text
Pattern：

七对
```

不得识别：

```text
豪华七对
```

Flower Calculation：

```text
七对

27张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

虽然手牌存在暗刻。

但本局还能胡其他牌。

因此不得判定豪华七对。

---

## RC-WIN-017 七对暗杠加成

Priority：

P0

Title：

暗杠组成两对参与七对。

Related Rules：

- SP-011

Action：

WIN

Purpose：

验证暗杠可作为两对参与七对。

Precondition：

玩家已经报听。

Input：

```text
手牌：

1111万（暗杠）

22万

33条

44条

55筒

66筒

7万

自摸：

7万
```

Expected：

服务器识别：

```text
Pattern：

七对
```

Flower Calculation：

```text
七对

27张花

+

暗杠组成两对

10张花

=

37张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

暗杠在七对中视为两个对子。

额外增加10张花。

---

## RC-WIN-018 清一色豪华七对

Priority：

P0

Title：

清一色豪华七对判定。

Related Rules：

- SP-011

Action：

WIN

Purpose：

验证清一色豪华七对识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111条

33条

44条

66条

77条

99条

自摸：

1条
```

Expected：

服务器识别：

```text
Pattern：

豪华七对

清一色
```

Flower Calculation：

```text
豪华七对

67张花

+

清一色

30张花

=

97张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

必须满足：

1、只能胡暗刻组成七对。

2、全部为同一花色。

最终花数为97张。

---

## RC-WIN-019 招招胡

Priority：

P0

Title：

招招胡判定。

Related Rules：

- SP-012

Action：

WIN

Purpose：

验证招招胡识别。

Precondition：

玩家已经报听。

Input：

```text
明杠：

444条

明杠：

555筒

手牌：

11万

22万

999筒

自摸：

2万
```

Expected：

服务器识别：

```text
Pattern：

招招胡
```

Flower Calculation：

```text
招招胡

46张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

所有刻子均为：

- 暗刻；
- 明杠；
- 暗杠。

符合招招胡定义。

---

## RC-WIN-020 清一色招招胡

Priority：

P0

Title：

清一色招招胡判定。

Related Rules：

- SP-013

Action：

WIN

Purpose：

验证清一色招招胡识别。

Precondition：

玩家已经报听。

Input：

```text
暗杠：

4444条

明杠：

5555条

手牌：

11条

22条

999条

自摸：

2条
```

Expected：

服务器识别：

```text
Pattern：

招招胡

清一色
```

Flower Calculation：

```text
招招胡

46张花

+

清一色

30张花

=

76张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

招招胡与清一色可以叠加。

---

## RC-WIN-021 对对清

Priority：

P0

Title：

对对清判定。

Related Rules：

- SP-014

Action：

WIN

Purpose：

验证清一色对对胡识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111条

222条

333条

44条

55条

自摸：

4条
```

Expected：

服务器识别：

```text
Pattern：

对对清
```

Flower Calculation：

```text
对对清

46张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

对对清即：

清一色 + 对对胡。

按照特色规则直接计46张花。

---

## RC-WIN-022 天外来客

Priority：

P0

Title：

天外来客判定。

Related Rules：

- SP-016

Action：

WIN

Purpose：

验证天外来客牌型识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

35条

123万

456万

789筒

11万

4条已被碰。
```

自摸：

```text
4条
```

Expected：

服务器识别：

```text
Pattern：

天外来客
```

Flower Calculation：

```text
天外来客

45张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

只有胡到已经被碰掉的丫子牌时。

才能判定天外来客。

---

## RC-WIN-023 全球独钓

Priority：

P0

Title：

全球独钓判定。

Related Rules：

- SP-017

Action：

WIN

Purpose：

验证全球独钓识别。

Precondition：

玩家已经报听。

Input：

```text
碰：

111万

碰：

222条

碰：

333筒

碰：

444万

手牌：

5条

自摸：

5条
```

Expected：

服务器识别：

```text
Pattern：

全球独钓
```

Flower Calculation：

```text
全球独钓

27张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

所有牌组均已固定。

手牌仅剩一张单吊。

符合全球独钓定义。

---

## RC-WIN-024 全球独钓招招胡

Priority：

P0

Title：

全球独钓招招胡判定。

Related Rules：

- SP-017

Action：

WIN

Purpose：

验证全球独钓与招招胡叠加。

Precondition：

玩家已经报听。

Input：

```text
明杠：

1111万

明杠：

2222条

暗杠：

3333筒

碰：

444万

手牌：

5条

自摸：

5条
```

Expected：

服务器识别：

```text
Pattern：

全球独钓

招招胡
```

Flower Calculation：

```text
全球独钓

27张花

+

招招胡

30张花

=

57张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

全球独钓形成招招胡。

按照规则额外增加30张花。

---

## RC-WIN-025 清一色全球独钓

Priority：

P0

Title：

清一色全球独钓判定。

Related Rules：

- SP-018

Action：

WIN

Purpose：

验证清一色全球独钓识别。

Precondition：

玩家已经报听。

Input：

```text
碰：

111条

碰：

222条

碰：

333条

碰：

444条

手牌：

5条

自摸：

5条
```

Expected：

服务器识别：

```text
Pattern：

全球独钓

清一色
```

Flower Calculation：

```text
全球独钓

27张花

+

清一色

30张花

=

57张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

全球独钓形成清一色。

按照规则增加30张花。

---

## RC-WIN-026 九莲宝灯

Priority：

P0

Title：

九莲宝灯判定。

Related Rules：

- SP-020

Action：

WIN

Purpose：

验证九莲宝灯识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

1112345678999条

自摸：

5条
```

Expected：

服务器识别：

```text
Pattern：

九莲宝灯
```

Flower Calculation：

```text
九莲宝灯

200张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

只要胡牌时最终牌型满足：

1112345678999

+

同花色任意一张。

均判定九莲宝灯。

---

## RC-WIN-027 非九莲宝灯

Priority：

P0

Title：

同一手牌不同胡法。

Related Rules：

- SP-020

Action：

WIN

Purpose：

验证并非所有胡法都属于九莲宝灯。

Precondition：

玩家已经报听。

Input：

```text
手牌：

1112345567899条

自摸：

6条
```

Expected：

服务器识别：

```text
Pattern：

普通清一色
```

不得识别：

```text
九莲宝灯
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

该手牌虽然可以听多张。

但只有摸到：

9条。

形成：

1112345678999

+

9条。

才属于九莲宝灯。

摸到其他可胡牌。

均按普通牌型计算。

---

## RC-WIN-028 幺幺胡

Priority：

P0

Title：

幺幺胡判定。

Related Rules：

- SP-021

Action：

WIN

Purpose：

验证幺幺胡识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111万

999万

111条

999条

11筒

自摸：

1筒

## RC-WIN-028 幺幺胡

Priority：

P0

Title：

幺幺胡判定。

Related Rules：

- SP-021

Action：

WIN

Purpose：

验证幺幺胡识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111万

999万

111条

999条

11筒

自摸：

1筒
```

Expected：

服务器识别：

```text
Pattern：

幺幺胡
```

Flower Calculation：

```text
幺幺胡

200张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

幺幺胡要求：

- 所有牌只能由 1 和 9 组成。
- 允许碰牌。
- 允许明杠。
- 允许暗杠。
- 允许补杠。
- 任意牌组中不得出现 2～8。

---

## RC-WIN-029 幺幺胡招招胡

Priority：

P0

Title：

幺幺胡招招胡判定。

Related Rules：

- SP-022

Action：

WIN

Purpose：

验证幺幺胡招招胡识别。

Precondition：

玩家已经报听。

Input：

```text
明杠：

1111万

暗杠：

9999万

碰：

111条

碰：

999条

手牌：

1筒

自摸：

1筒
```

Expected：

服务器识别：

```text
Pattern：

幺幺胡

招招胡
```

Flower Calculation：

```text
幺幺胡

200张花

+

招招胡

100张花

=

300张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

符合幺幺胡条件。

所有刻子均为：

- 明杠
- 暗杠
- 碰牌

因此形成：

幺幺胡招招胡。

---

## RC-WIN-030 非幺幺胡

Priority：

P0

Title：

存在中张不得判定幺幺胡。

Related Rules：

- SP-021

Action：

WIN

Purpose：

验证幺幺胡限制。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111万

999万

111条

789条

11筒

自摸：

9筒
```

Expected：

服务器识别：

```text
Pattern：

普通胡牌
```

不得识别：

```text
幺幺胡
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

只要牌组中出现任意 2～8 的牌。

即不符合幺幺胡定义。

不得按幺幺胡计分。

---

## RC-WIN-031 天胡

Priority：

P0

Title：

天胡事件判定。

Related Rules：

- GR-046
- GR-051

Action：

WIN

Purpose：

验证天胡事件识别。

Precondition：

玩家处于开局阶段。

补花结束立即形成合法胡牌。

Input：

```text
Current Phase：

INITIAL_BU_HUA

Can Win：

true
```

Expected：

服务器识别：

```text
Event：

TIAN_HU
```

Flower Calculation：

```text
天胡

100张花
```

不得识别：

```text
HAI_DI

GANG_KAI
```

Broadcast：

```text
WIN_SUCCESS

EVENT_TIAN_HU
```

Notes：

天胡属于胡牌事件。

不得与其它胡牌事件同时成立。

---

## RC-WIN-032 海底捞月

Priority：

P0

Title：

海底捞月事件判定。

Related Rules：

- GR-050

Action：

WIN

Purpose：

验证海底捞月识别。

Precondition：

玩家摸到本局最后一张有效牌。

形成合法胡牌。

Input：

```text
Current Draw：

LAST_VALID_TILE
```

Expected：

服务器识别：

```text
Event：

HAI_DI
```

Flower Calculation：

```text
海底捞月

10张花
```

不得识别：

```text
TIAN_HU
```

Broadcast：

```text
WIN_SUCCESS

EVENT_HAI_DI
```

Notes：

最后一张有效牌包括：

- 普通摸牌；
- 杠尾摸牌；
- 补花摸牌。

---

## RC-WIN-033 杠开

Priority：

P0

Title：

杠开事件判定。

Related Rules：

- GR-049

Action：

WIN

Purpose：

验证杠开识别。

Precondition：

玩家杠牌后。

从杠尾摸牌形成合法胡牌。

Input：

```text
Draw Source：

GANG_DRAW
```

Expected：

服务器识别：

```text
Event：

GANG_KAI
```

Flower Calculation：

```text
杠开

2张花
```

不得识别：

```text
TIAN_HU
```

Broadcast：

```text
WIN_SUCCESS

EVENT_GANG_KAI
```

Notes：

包括：

- 明杠；
- 补杠；
- 暗杠；
- 补花后杠尾摸牌。

均属于杠开。

---

## RC-WIN-034 海底捞月与杠开同时成立

Priority：

P0

Title：

海底捞月与杠开叠加判定。

Related Rules：

- GR-049
- GR-050
- GR-051

Action：

WIN

Purpose：

验证海底捞月与杠开允许同时成立。

Precondition：

玩家因杠牌或补花。

摸到本局最后一张有效牌。

形成合法胡牌。

Input：

```text
Draw Source：

GANG_DRAW

Current Draw：

LAST_VALID_TILE
```

Expected：

服务器识别：

```text
Event：

HAI_DI

GANG_KAI
```

Flower Calculation：

```text
海底捞月

10张花

+

杠开

2张花

=

12张花
```

Broadcast：

```text
WIN_SUCCESS

EVENT_HAI_DI

EVENT_GANG_KAI
```

Notes：

海底捞月与杠开允许同时成立。

属于两个独立事件。

不得互相覆盖。

---

## RC-WIN-035 单边大绝

Priority：P0

Title：单边大绝判定。

Related Rules：AD-01

Action：WIN

Purpose：验证单边大绝识别。

Precondition：玩家已经报听，可胡多张，摸到其中一张绝张。

Input：

```text
手牌：

23条

123万

456万

789筒

11万

可胡：1条、4条

其中1条已被其他玩家碰光（公开碰牌可见）。

自摸：1条
```

Expected：

服务器识别：

```text
Bonus：单边大绝
```

Flower Calculation：

```text
正常胡牌花数 + 单边大绝 +10
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

单边大绝要求：

1、能胡多张（本例可胡1条或4条）。

2、实际摸到的那张是绝张。

3、绝张判定仅依据公开碰牌和明杠，不统计其他玩家手中暗刻。

若本例只能胡4条（丫子），且4条被碰光，则应判定为天外来客，不得判定为单边大绝。

---

## RC-WIN-036 双边大绝

Priority：P0

Title：双边大绝判定。

Related Rules：AD-02

Action：WIN

Purpose：验证双边大绝识别。

Precondition：玩家已经报听，有且仅胡两张，两张均已绝张。

Input：

```text
手牌：

23条

123万

456万

789筒

11万

可胡：1条、4条（仅此两张）

1条已被碰光，4条已被碰光。

自摸：4条
```

Expected：

服务器识别：

```text
Bonus：双边大绝
```

Flower Calculation：

```text
正常胡牌花数 + 双边大绝 +20
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

双边大绝要求：

1、有且仅听两张（唯一性）。

2、两张均已绝张（公开可见）。

3、摸到任意一张均算双边大绝。

若还能胡第三张，则不得判定双边大绝，只能按单边大绝或无大绝判定。

---

## RC-WIN-037 三边大绝

Priority：P0

Title：三边大绝判定。

Related Rules：AD-03

Action：WIN

Purpose：验证三边大绝识别。

Precondition：玩家已经报听，有且仅胡三张，三张均已绝张。

Input：

```text
手牌：

23456条

123万

789万

11筒

可胡：1条、4条、7条（仅此三张）

1条、4条、7条均已绝张（公开可见）。

自摸：4条
```

Expected：

服务器识别：

```text
Bonus：三边大绝
```

Flower Calculation：

```text
正常胡牌花数 + 三边大绝 +30
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

三边大绝要求：

1、有且仅听三张（唯一性）。

2、三张均已绝张（公开可见）。

3、摸到任意一张均算三边大绝。

若还能胡第四张，则不得判定三边大绝。

---

## RC-WIN-035B 天外来客与单边大绝的区别

Priority：P0

Title：天外来客不得判定为单边大绝。

Related Rules：SP-016，AD-01

Action：WIN

Purpose：防止混淆天外来客与单边大绝。

Input：

```text
手牌：

35条

123万

456万

789筒

11万

可胡：4条（仅此一张，丫子牌型）

4条已被碰光。

自摸：4条
```

Expected：

服务器识别：

```text
Pattern：天外来客
```

Flower Calculation：

```text
天外来客 45张花
```

不得识别：

```text
Bonus：单边大绝
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

天外来客定义：

丫子牌型（只能胡唯一一张），且该张已被碰光，摸到胡牌。

天外来客是独立牌型，不与大绝叠加。

单边大绝必须满足：能胡多张，摸到其中绝张。

两者互斥，不得同时成立。

## RC-WIN-038 暗绝

Priority：

P0

Title：

暗绝判定。

Related Rules：

- SP-026

Action：

WIN

Purpose：

验证暗绝识别。

Precondition：

玩家已经报听。

Input：

```text
手牌：

333条

45条

11万

222万

789筒

自摸：

3条
```

Expected：

服务器识别：

```text
Bonus：

暗绝
```

Flower Calculation：

```text
暗绝

10张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

玩家手中已有暗刻。

胡该暗刻组成刻子。

形成暗绝。

---

## RC-WIN-039 不满足暗绝

Priority：

P0

Title：

暗绝限制验证。

Related Rules：

- SP-026

Action：

WIN

Purpose：

验证暗绝必须胡暗刻。

Precondition：

玩家已经报听。

Input：

```text
手牌：

333条

45条

11万

222万

789筒

自摸：

6条
```

Expected：

服务器不得识别：

```text
Bonus：

暗绝
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

虽然手牌存在暗刻。

但本次胡牌并非胡暗刻。

不得判定暗绝。

---

## RC-WIN-040 清一色七对 + 暗杠加成

Priority：

P0

Title：

清一色七对暗杠加成判定。

Related Rules：

- SP-010
- SP-011

Action：

WIN

Purpose：

验证清一色七对与暗杠加成同时成立。

Precondition：

玩家已经报听。

Input：

```text
暗杠：

1111条

手牌：

22条

33条

44条

55条

66条

7条

自摸：

7条
```

Expected：

服务器识别：

```text
Pattern：

七对

清一色
```

Bonus：

```text
暗杠组成两对
```

Flower Calculation：

```text
七对

27张花

+

清一色

30张花

+

暗杠两对

10张花

=

67张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

暗杠在七对中按两个对子计算。

---

## RC-WIN-041 豪华七对清一色

Priority：

P0

Title：

豪华七对清一色判定。

Related Rules：

- SP-011

Action：

WIN

Purpose：

验证豪华七对与清一色组合。

Precondition：

玩家已经报听。

Input：

```text
手牌：

111条

33条

44条

66条

77条

99条

自摸：

1条
```

Expected：

服务器识别：

```text
Pattern：

豪华七对

清一色
```

Flower Calculation：

```text
豪华七对

67张花

+

清一色

30张花

=

97张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

只能胡暗刻组成七对。

否则不得判定豪华七对。

---

## RC-WIN-042 全球独钓招招清

Priority：

P0

Title：

全球独钓招招清判定。

Related Rules：

- SP-017

Action：

WIN

Purpose：

验证全球独钓、招招胡、清一色组合。

Precondition：

玩家已经报听。

Input：

```text
明杠：

1111条

暗杠：

2222条

碰：

333条

碰：

444条

手牌：

5条

自摸：

5条
```

Expected：

服务器识别：

```text
Pattern：

全球独钓

招招胡

清一色
```

Flower Calculation：

```text
全球独钓

27张花

+

招招胡

30张花

+

清一色

30张花

=

87张花
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

按照规则：

全球独钓形成招招胡加30张。

再形成清一色继续加30张。

最终87张花。

---

## RC-WIN-043 放弃普通胡后胡九莲宝灯

Priority：

P0

Title：

放弃普通胡后胡九莲宝灯。

Related Rules：

- GR-036
- SP-020

Action：

WIN

Purpose：

验证玩家可放弃普通胡，后续胡更大牌型。

Precondition：

玩家已经报听。

玩家第一次摸牌可普通胡。

主动选择：

PASS。

继续游戏。

再次摸牌。

形成九莲宝灯。

Expected：

服务器第一次：

```text
WIN_AVAILABLE

PLAYER_PASS_WIN
```

第二次：

服务器识别：

```text
Pattern：

九莲宝灯
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

服务器只根据本次胡牌时手牌判定。

之前放弃胡牌。

不影响后续牌型。

---

## RC-WIN-044 放弃普通胡后胡豪华七对

Priority：

P0

Title：

放弃普通胡后胡豪华七对。

Related Rules：

- GR-036
- SP-011

Action：

WIN

Purpose：

验证放弃普通胡后可形成豪华七对。

Precondition：

玩家已经报听。

第一次摸牌。

形成普通七对。

选择：

PASS。

继续摸牌。

第二次摸牌。

形成豪华七对。

Expected：

服务器最终识别：

```text
Pattern：

豪华七对
```

Broadcast：

```text
WIN_SUCCESS
```

Notes：

最终胡牌牌型。

覆盖之前放弃的普通牌型。

---

## RC-WIN-045 放弃普通胡后胡更高番

Priority：

P0

Title：

放弃小牌后胡大牌。

Related Rules：

- GR-036

Action：

WIN

Purpose：

验证服务器始终按最终胡牌牌型结算。

Precondition：

玩家存在多个可胡机会。

前一次放弃。

最后一次选择胡牌。

Expected：

服务器：

仅识别最终胡牌。

不得记录：

第一次放弃胡牌。

Broadcast：

```text
WIN_SUCCESS
```

Notes：

服务器不得缓存之前的胡牌结果。

所有计算均以最终胡牌时手牌为准。

---

## RC-WIN-046 胡牌识别完成

Priority：

P1

Title：

WIN模块结束。

Related Rules：

- GR-039

Action：

WIN

Purpose：

验证服务器完成全部胡牌识别。

Expected：

服务器输出：

```text
BaseType

Pattern

Modifier

Event

Bonus
```

进入：

```text
SETTLEMENT
```

Broadcast：

```text
START_SETTLEMENT
```

Notes：

WIN模块不负责最终花数累加。

最终花数由SETTLEMENT模块统一计算。

---

---

# 十四、SETTLEMENT（结算）

服务器收到：

SETTLEMENT

必须按照固定顺序计算最终花数。

计算顺序：

1、基础胡型（Base Type）

2、特殊牌型（Pattern）

3、清一色加成（Modifier）

4、胡牌事件（Event）

5、奖励项（Bonus）

6、最终花数（Total Flower）

服务器不得改变 WIN 模块已经识别出的任何结果。

SETTLEMENT 只负责计算。

---

## RC-SETTLEMENT-001 普通多张

Priority：

P0

Title：

普通多张花数。

Related Rules：

- SP-001

Action：

SETTLEMENT

Purpose：

验证普通多张结算。

Input：

```text
Base Type：

多张
```

Expected：

```text
Base Type：

3

Pattern：

0

Modifier：

0

Event：

0

Bonus：

0

Total：

3
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通多张无任何叠加。

---

## RC-SETTLEMENT-002 清一色多张

Priority：

P0

Title：

清一色多张结算。

Related Rules：

- SP-001

Action：

SETTLEMENT

Purpose：

验证清一色加成。

Input：

```text
Base Type：

多张

Modifier：

清一色
```

Expected：

```text
Base Type：

3

Modifier：

30

Total：

33
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

清一色统一增加30张。

---

## RC-SETTLEMENT-003 丫子

Priority：

P0

Title：

普通丫子结算。

Related Rules：

- SP-002

Action：

SETTLEMENT

Purpose：

验证丫子结算。

Input：

```text
Base Type：

丫子
```

Expected：

```text
Base Type：

5

Pattern：

0

Modifier：

0

Event：

0

Bonus：

0

Total：

5
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通丫子无其它加成。

---

## RC-SETTLEMENT-004 清一色丫子

Priority：

P0

Title：

清一色丫子结算。

Related Rules：

- SP-002

Action：

SETTLEMENT

Purpose：

验证清一色丫子结算。

Input：

```text
Base Type：

丫子

Modifier：

清一色
```

Expected：

```text
Base Type：

5

Pattern：

0

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

35
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

清一色统一增加30张花。

---

## RC-SETTLEMENT-005 对倒

Priority：

P0

Title：

普通对倒结算。

Related Rules：

- SP-003

Action：

SETTLEMENT

Purpose：

验证普通对倒结算。

Input：

```text
Base Type：

对倒
```

Expected：

```text
Base Type：

6

Pattern：

0

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

6
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通对倒无其它加成。

---

## RC-SETTLEMENT-006 清一色对倒

Priority：

P0

Title：

清一色对倒结算。

Related Rules：

- SP-003

Action：

SETTLEMENT

Purpose：

验证清一色对倒结算。

Input：

```text
Base Type：

对倒

Modifier：

清一色
```

Expected：

```text
Base Type：

6

Pattern：

0

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

36
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通对倒形成清一色。

额外增加30张花。

---

## RC-SETTLEMENT-007 单吊

Priority：

P0

Title：

普通单吊结算。

Related Rules：

- SP-004

Action：

SETTLEMENT

Purpose：

验证普通单吊结算。

Input：

```text
Base Type：

单吊
```

Expected：

```text
Base Type：

7

Pattern：

0

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

7
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通单吊无其它加成。

---

## RC-SETTLEMENT-008 清一色单吊

Priority：

P0

Title：

清一色单吊结算。

Related Rules：

- SP-004

Action：

SETTLEMENT

Purpose：

验证清一色单吊结算。

Input：

```text
Base Type：

单吊

Modifier：

清一色
```

Expected：

```text
Base Type：

7

Pattern：

0

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

37
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

清一色统一增加30张花。

---

## RC-SETTLEMENT-009 拐头丫子

Priority：

P0

Title：

拐头丫子结算。

Related Rules：

- SP-005

Action：

SETTLEMENT

Purpose：

验证拐头丫子结算。

Input：

```text
Base Type：

拐头丫子
```

Expected：

```text
Base Type：

15

Pattern：

0

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

15
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通拐头丫子无其它加成。

---

## RC-SETTLEMENT-010 清一色拐头丫子

Priority：

P0

Title：

清一色拐头丫子结算。

Related Rules：

- SP-005

Action：

SETTLEMENT

Purpose：

验证清一色拐头丫子结算。

Input：

```text
Base Type：

拐头丫子

Modifier：

清一色
```

Expected：

```text
Base Type：

15

Pattern：

0

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

45
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

拐头丫子形成清一色。

额外增加30张花。

---

## RC-SETTLEMENT-011 对倒对对胡

Priority：

P0

Title：

对倒对对胡结算。

Related Rules：

- SP-006

Action：

SETTLEMENT

Purpose：

验证对倒对对胡最终结算。

Input：

```text
Base Type：

对倒

Pattern：

对对胡
```

Expected：

```text
Base Type：

6

Pattern：

10

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

16
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

对倒对对胡 = 对倒（6）+ 对对胡（10）。

---

## RC-SETTLEMENT-012 单吊对对胡

Priority：

P0

Title：

单吊对对胡结算。

Related Rules：

- SP-007

Action：

SETTLEMENT

Purpose：

验证单吊对对胡最终结算。

Input：

```text
Base Type：

单吊

Pattern：

对对胡
```

Expected：

```text
Base Type：

7

Pattern：

10

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

17
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

单吊对对胡 = 单吊（7）+ 对对胡（10）。

---

## RC-SETTLEMENT-013 多张对对胡

Priority：

P0

Title：

多张对对胡结算。

Related Rules：

- SP-008

Action：

SETTLEMENT

Purpose：

验证多张对对胡最终结算。

Input：

```text
Base Type：

多张

Pattern：

对对胡
```

Expected：

```text
Base Type：

3

Pattern：

10

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

13
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

多张对对胡 = 多张（3）+ 对对胡（10）。

---

## RC-SETTLEMENT-014 七对

Priority：

P0

Title：

七对结算。

Related Rules：

- SP-009

Action：

SETTLEMENT

Purpose：

验证七对最终结算。

Input：

```text
Pattern：

七对
```

Expected：

```text
Base Type：

0

Pattern：

27

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

27
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通七对最终结算27张花。

---

## RC-SETTLEMENT-015 清一色七对

Priority：

P0

Title：

清一色七对结算。

Related Rules：

- SP-010

Action：

SETTLEMENT

Purpose：

验证清一色七对最终结算。

Input：

```text
Pattern：

七对

Modifier：

清一色
```

Expected：

```text
Base Type：

0

Pattern：

27

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

57
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

清一色统一增加30张花。

---

## RC-SETTLEMENT-016 豪华七对

Priority：

P0

Title：

豪华七对结算。

Related Rules：

- SP-011

Action：

SETTLEMENT

Purpose：

验证豪华七对最终结算。

Input：

```text
Pattern：

豪华七对
```

Expected：

```text
Base Type：

0

Pattern：

67

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

67
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

豪华七对最终结算67张花。

---

## RC-SETTLEMENT-017 清一色豪华七对

Priority：

P0

Title：

清一色豪华七对结算。

Related Rules：

- SP-011

Action：

SETTLEMENT

Purpose：

验证清一色豪华七对最终结算。

Input：

```text
Pattern：

豪华七对

Modifier：

清一色
```

Expected：

```text
Base Type：

0

Pattern：

67

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

97
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

清一色豪华七对最终结算97张花。

---

## RC-SETTLEMENT-018 七对暗杠加成

Priority：

P0

Title：

七对暗杠加成结算。

Related Rules：

- SP-011

Action：

SETTLEMENT

Purpose：

验证七对暗杠加成最终结算。

Input：

```text
Pattern：

七对

Bonus：

暗杠组成两对
```

Expected：

```text
Base Type：

None

Pattern：

27

Modifier：

0

Event：

0

Bonus：

10

----------------

Total：

37
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

七对中暗杠按两个对子计算。

额外增加10张花。

---

## RC-SETTLEMENT-019 清一色七对暗杠加成

Priority：

P0

Title：

清一色七对暗杠加成结算。

Related Rules：

- SP-011

Action：

SETTLEMENT

Purpose：

验证清一色七对暗杠加成最终结算。

Input：

```text
Pattern：

七对

Modifier：

清一色

Bonus：

暗杠组成两对
```

Expected：

```text
Base Type：

None

Pattern：

27

Modifier：

30

Event：

0

Bonus：

10

----------------

Total：

67
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

七对 + 清一色 + 暗杠组成两对。

最终67张花。

---

## RC-SETTLEMENT-020 招招胡

Priority：

P0

Title：

招招胡结算。

Related Rules：

- SP-012

Action：

SETTLEMENT

Purpose：

验证招招胡最终结算。

Input：

```text
Pattern：

招招胡
```

Expected：

```text
Base Type：

None

Pattern：

46

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

46
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

普通招招胡最终结算46张花。

---

## RC-SETTLEMENT-021 招招清

Priority：

P0

Title：

招招清结算。

Related Rules：

- SP-015

Action：

SETTLEMENT

Purpose：

验证招招清最终结算。

Input：

```text
Pattern：

招招胡

Modifier：

清一色
```

Expected：

```text
Base Type：

None

Pattern：

46

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

76
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

招招胡形成清一色。

统一称为：

招招清。

最终76张花。

---

## RC-SETTLEMENT-022 对对清

Priority：

P0

Title：

对对清结算。

Related Rules：

- SP-014

Action：

SETTLEMENT

Purpose：

验证对对清最终结算。

Input：

```text
Base Type：

对倒对对胡

Modifier：

清一色
```

Expected：

```text
Base Type：

16

Pattern：

0

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

46
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

对对清即清一色对对胡。

最终46张花。

---

## RC-SETTLEMENT-023 天外来客

Priority：

P0

Title：

天外来客结算。

Related Rules：

- SP-016

Action：

SETTLEMENT

Purpose：

验证天外来客最终结算。

Input：

```text
Pattern：

天外来客
```

Expected：

```text
Base Type：

None

Pattern：

45

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

45
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

天外来客固定45张花。

---

## RC-SETTLEMENT-024 全球独钓

Priority：

P0

Title：

全球独钓结算。

Related Rules：

- SP-017

Action：

SETTLEMENT

Purpose：

验证全球独钓最终结算。

Input：

```text
Pattern：

全球独钓
```

Expected：

```text
Base Type：

None

Pattern：

27

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

27
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

全球独钓固定27张花。

---

## RC-SETTLEMENT-025 清一色全球独钓

Priority：

P0

Title：

清一色全球独钓结算。

Related Rules：

- SP-018

Action：

SETTLEMENT

Purpose：

验证清一色全球独钓最终结算。

Input：

```text
Pattern：

全球独钓

Modifier：

清一色
```

Expected：

```text
Base Type：

None

Pattern：

27

Modifier：

30

Event：

0

Bonus：

0

----------------

Total：

57
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

全球独钓形成清一色。

最终57张花。

---

## RC-SETTLEMENT-026 天胡

Priority：

P0

Title：

天胡结算。

Related Rules：

- SP-019

Action：

SETTLEMENT

Purpose：

验证天胡事件最终结算。

Input：

```text
Event：

天胡
```

Expected：

```text
Base Type：

None

Pattern：

0

Modifier：

0

Event：

100

Bonus：

0

----------------

Total：

100
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

天胡属于事件加成。

固定增加100张花。

---

## RC-SETTLEMENT-027 九莲宝灯

Priority：

P0

Title：

九莲宝灯结算。

Related Rules：

- SP-020

Action：

SETTLEMENT

Purpose：

验证九莲宝灯最终结算。

Input：

```text
Pattern：

九莲宝灯
```

Expected：

```text
Base Type：

None

Pattern：

200

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

200
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

九莲宝灯固定200张花。

不得再叠加清一色。

---

## RC-SETTLEMENT-028 幺幺胡

Priority：

P0

Title：

幺幺胡结算。

Related Rules：

- SP-021

Action：

SETTLEMENT

Purpose：

验证幺幺胡最终结算。

Input：

```text
Pattern：

幺幺胡
```

Expected：

```text
Base Type：

None

Pattern：

200

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

200
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

幺幺胡固定200张花。

不得叠加清一色。

---

## RC-SETTLEMENT-029 幺幺胡招招胡

Priority：

P0

Title：

幺幺胡招招胡结算。

Related Rules：

- SP-022

Action：

SETTLEMENT

Purpose：

验证幺幺胡招招胡最终结算。

Input：

```text
Pattern：

幺幺胡招招胡
```

Expected：

```text
Base Type：

None

Pattern：

300

Modifier：

0

Event：

0

Bonus：

0

----------------

Total：

300
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

幺幺胡招招胡固定300张花。

不得再叠加清一色。

---

## RC-SETTLEMENT-030 七对 + 海底捞月

Priority：

P0

Title：

七对海底捞月结算。

Related Rules：

- SP-009
- GR-050

Action：

SETTLEMENT

Purpose：

验证七对与海底捞月叠加。

Input：

```text
Pattern：

七对

Event：

海底捞月
```

Expected：

```text
Pattern：

27

Event：

10

Bonus：

0

----------------

Total：

37
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

海底捞月正常叠加。

---

## RC-SETTLEMENT-031 七对 + 杠开

Priority：

P0

Title：

七对杠开结算。

Related Rules：

- SP-009
- GR-049

Action：

SETTLEMENT

Purpose：

验证七对杠开叠加。

Input：

```text
Pattern：

七对

Event：

杠开
```

Expected：

```text
Pattern：

27

Event：

2

Bonus：

0

----------------

Total：

29
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

杠开正常增加2张花。

---

## RC-SETTLEMENT-032 七对 + 海底 + 杠开

Priority：

P0

Title：

七对海底杠开结算。

Related Rules：

- SP-009
- GR-049
- GR-050

Action：

SETTLEMENT

Purpose：

验证海底与杠开共同成立。

Input：

```text
Pattern：

七对

Event：

海底捞月

杠开
```

Expected：

```text
Pattern：

27

Event：

12

Bonus：

0

----------------

Total：

39
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

海底10。

杠开2。

允许同时成立。

---

## RC-SETTLEMENT-033 清一色七对 + 海底 + 杠开

Priority：

P0

Title：

清一色七对海底杠开。

Related Rules：

- SP-010
- GR-049
- GR-050

Action：

SETTLEMENT

Purpose：

验证多个模块共同结算。

Input：

```text
Pattern：

七对

Modifier：

清一色

Event：

海底捞月

杠开
```

Expected：

```text
Pattern：

27

Modifier：

30

Event：

12

Bonus：

0

----------------

Total：

69
```

Broadcast：

```text
SETTLEMENT_SUCCESS
```

Notes：

这是综合结算测试。

验证多个模块共同参与计算。

---

---

# 十五、REGRESSION（回归测试）

REGRESSION 用于验证：

历史 Bug。

特殊规则。

容易误判规则。

任何修复后的版本。

必须全部通过。

---

## RC-REGRESSION-001 豪华七对唯一胡牌限制

Priority：

P0

Title：

豪华七对必须唯一胡牌。

Related Rules：

- SP-011

Purpose：

防止豪华七对误判。

Input：

```text
手牌：

1112233445566
```

Expected：

服务器不得识别：

```text
豪华七对
```

Reason：

该手牌除暗刻外。

仍可胡其它牌。

Notes：

豪华七对必须：

只能胡暗刻组成七对。

---

## RC-REGRESSION-002 豪华七对正确判定

Priority：

P0

Title：

唯一胡牌形成豪华七对。

Related Rules：

- SP-011

Purpose：

验证豪华七对正确识别。

Input：

```text
手牌：

1113344667799
```

Expected：

服务器识别：

```text
豪华七对
```

Reason：

只能胡：

1万。

形成豪华七对。

---

## RC-REGRESSION-003 九莲宝灯唯一胡法

Priority：

P0

Title：

九莲宝灯限制。

Related Rules：

- SP-020

Purpose：

验证九莲宝灯不得误判。

Input：

```text
手牌：

1112345567899条
```

Expected：

摸：

```text
9条
```

服务器识别：

```text
九莲宝灯
```

摸：

```text
3条

6条
```

服务器不得识别：

```text
九莲宝灯
```

Notes：

最终胡牌牌型决定九莲宝灯。

---

## RC-REGRESSION-004 放弃普通胡后胡九莲宝灯

Priority：

P0

Title：

放弃小牌后胡大牌。

Related Rules：

- GR-036

Purpose：

验证服务器重新判定牌型。

Process：

第一次：

普通清一色。

玩家：

PASS。

第二次：

九莲宝灯。

Expected：

最终：

```text
九莲宝灯
```

不得记录：

第一次普通胡牌。

---

## RC-REGRESSION-005 天胡不得判定杠开

Priority：

P0

Title：

天胡与杠开互斥。

Related Rules：

- GR-046
- GR-049

Purpose：

验证天胡不得同时判定杠开。

Expected：

服务器识别：

```text
Event：

TIAN_HU
```

不得识别：

```text
GANG_KAI
```

---

## RC-REGRESSION-006 天胡不得判定海底捞月

Priority：

P0

Title：

天胡与海底互斥。

Related Rules：

- GR-046
- GR-050

Purpose：

验证天胡不得同时判定海底。

Expected：

服务器识别：

```text
Event：

TIAN_HU
```

不得识别：

```text
HAI_DI
```

---

## RC-REGRESSION-007 海底允许杠开

Priority：

P0

Title：

海底与杠开允许同时成立。

Related Rules：

- GR-049
- GR-050

Purpose：

验证海底杠开组合。

Expected：

服务器识别：

```text
HAI_DI

GANG_KAI
```

---

## RC-REGRESSION-008 报听后允许杠牌

Priority：

P0

Title：

报听后允许合法杠牌。

Related Rules：

- GR-032

Purpose：

验证报听后合法杠牌。

Expected：

保持听牌。

允许继续杠牌。

---

## RC-REGRESSION-009 报听后失听禁止杠牌

Priority：

P0

Title：

报听后禁止失听。

Related Rules：

- GR-032

Purpose：

验证报听后杠牌导致失听。

Expected：

服务器拒绝杠牌。

---

## RC-REGRESSION-010 可以放弃胡牌

Priority：

P0

Title：

玩家允许放弃胡牌。

Related Rules：

- GR-036

Purpose：

验证PASS。

Expected：

服务器继续游戏。

不得自动胡牌。

---

## RC-REGRESSION-011 七对暗杠按两对计算

Priority：

P0

Title：

七对暗杠规则。

Related Rules：

- SP-011

Purpose：

验证暗杠作为两个对子。

Expected：

增加：

10张花。

---

## RC-REGRESSION-012 招招清统一名称

Priority：

P1

Title：

统一牌型名称。

Purpose：

验证服务器统一输出。

Expected：

服务器输出：

```text
招招清
```

不得输出：

```text
清一色招招胡
```

---

## RC-REGRESSION-013 全球独钓固定27张

Priority：

P0

Title：

全球独钓花数。

Purpose：

验证最新规则。

Expected：

```text
27张花
```

不得出现：

```text
35张花
```

---

## RC-REGRESSION-014 清一色七对57张

Priority：

P0

Title：

清一色七对修正规则。

Purpose：

验证修正后的花数。

Expected：

```text
57张花
```

不得出现：

```text
47张花
```

---

## RC-REGRESSION-015 九莲宝灯不得叠加清一色

Priority：

P0

Title：

九莲宝灯固定花数。

Purpose：

验证固定200张。

Expected：

```text
200张花
```

不得计算：

```text
230张花
```

---

## RC-REGRESSION-016 幺幺胡不得叠加清一色

Priority：

P0

Title：

幺幺胡固定花数。

Purpose：

验证固定200张。

Expected：

```text
200张花
```

不得计算：

```text
230张花
```

---

## RC-REGRESSION-017 幺幺胡招招胡固定300张

Priority：

P0

Title：

幺幺胡招招胡。

Purpose：

验证固定300张。

Expected：

```text
300张花
```

---

## RC-REGRESSION-018 双边大绝限制

Priority：

P0

Title：

双边大绝必须只有两个听口。

Purpose：

验证双边大绝。

Expected：

若存在第三个可胡牌。

不得判定：

```text
双边大绝
```

---

## RC-REGRESSION-019 三边大绝限制

Priority：

P0

Title：

三边大绝必须只有三个听口。

Purpose：

验证三边大绝。

Expected：

听牌列表必须仅包含三个听口。

---

## RC-REGRESSION-020 补花结束属于天胡

Priority：

P0

Title：

补花结束立即胡牌。

Purpose：

验证补花阶段天胡。

Expected：

服务器识别：

```text
TIAN_HU
```

---

## RC-REGRESSION-021 庄家无花直接天胡

Priority：

P0

Title：

庄家无花。

Purpose：

验证庄家无花天胡。

Expected：

服务器识别：

```text
TIAN_HU
```

---

## RC-REGRESSION-022 闲家补花也可天胡

Priority：

P0

Title：

闲家补花。

Purpose：

验证特色规则。

Expected：

服务器识别：

```text
TIAN_HU
```

---

## RC-REGRESSION-023 最终胡牌覆盖之前胡牌

Priority：

P0

Title：

最终牌型原则。

Purpose：

验证最终胡牌。

Expected：

服务器只计算：

最终胡牌。

不得记录：

之前PASS胡牌。

---

## RC-REGRESSION-024 所有结算以03为准

Priority：

P0

Title：

统一结算来源。

Purpose：

验证服务器结算来源。

Expected：

所有花数。

全部来自：

03_Scoring_Table.md

不得出现硬编码。

---
