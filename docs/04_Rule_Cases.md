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
