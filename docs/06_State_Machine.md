# 天长麻将微信小程序（Tianchang Mahjong Mini Program）

# 06_State_Machine

Version：1.0.0

Status：Draft

Author：Shawn Hsu & Claude

---

# 一、文档目的

本文档定义天长麻将服务器的完整游戏状态机（State Machine）。

所有服务器状态流转必须严格遵守本文档。

不得在状态机之外处理任何游戏逻辑。

---

# 二、设计原则

1. 服务器拥有唯一状态，客户端无状态。
2. 每个状态只允许处理特定消息，其余消息一律拒绝。
3. 状态流转必须是单向的，不得跳跃。
4. 任何异常必须回退到上一个合法状态。
5. 状态变化必须广播给所有玩家。

---

# 三、状态总览

```
IDLE
  ↓
WAIT_JOIN
  ↓
WAIT_READY
  ↓
DEALING
  ↓
INITIAL_BU_HUA
  ↓
WAIT_DISCARD  ←──────────────────────────────────────────┐
  ↓                                                       │
WAIT_RESPONSE                                             │
  ↓                    ↓                    ↓             │
WAIT_DRAW         WAIT_PENG_DISCARD   WAIT_GANG_DRAW      │
  ↓                    ↓                    ↓             │
WAIT_ACTION ──────────────────────────────────────────────┘
  ↓
SETTLEMENT
  ↓
ROUND_END
  ↓
WAIT_READY（下一局）
或
ROOM_CLOSED
```

---

# 四、状态定义

## S-001 IDLE

**说明：**

房间刚创建，尚无玩家加入。

**允许操作：**

- ROOM_JOIN

**禁止操作：**

- 所有游戏操作

**流转条件：**

第一个玩家加入 → WAIT_JOIN

---

## S-002 WAIT_JOIN

**说明：**

等待玩家加入，人数不足4人。

**允许操作：**

- ROOM_JOIN
- ROOM_DISMISS

**禁止操作：**

- 所有游戏操作
- PLAYER_READY

**流转条件：**

4名玩家全部加入 → WAIT_READY

---

## S-003 WAIT_READY

**说明：**

4名玩家已全部加入，等待全部准备。

**允许操作：**

- PLAYER_READY
- ROOM_DISMISS

**禁止操作：**

- 所有游戏操作

**流转条件：**

4名玩家全部准备 → DEALING

---

## S-004 DEALING

**说明：**

服务器正在洗牌、发牌。

持续时间极短，客户端不需要等待操作。

**服务器执行：**

1. 洗牌（136张）
2. 掷骰子，确定跳牌位置
3. 发牌（庄家14张，其余13张）
4. 初始化花牌区、弃牌区、碰杠区

**流转条件：**

发牌完成 → INITIAL_BU_HUA

---

## S-005 INITIAL_BU_HUA

**说明：**

开局补花阶段。

按照庄家→南家→西家→北家顺序依次补花。

每位玩家补花完成后，检查是否天胡。

**服务器执行：**

对每位玩家（按顺序）：

```
检查手牌是否有基本花
  ↓
有 → 移入花牌区 → 从杠尾补牌
       ↓
     继续检查是否有基本花
       ↓
     无 → 检查是否可胡（天胡判定）
            ↓
          可胡 → 触发 CAN_WIN（等待玩家决定）
          不可胡 → 轮到下一位玩家
```

**允许操作：**

- WIN（天胡）
- PASS（放弃天胡）

**禁止操作：**

- DISCARD
- PENG
- GANG（任何杠牌）
- TING

**流转条件：**

所有玩家补花完成且无天胡 → WAIT_DISCARD（庄家出牌）

任意玩家天胡 → SETTLEMENT

---

## S-006 WAIT_DISCARD

**说明：**

当前行动玩家拥有出牌权，等待打出一张牌。

**当前行动玩家允许操作：**

- DISCARD（出牌）
- WIN（若服务器判定可胡）
- AN_GANG（若服务器判定可暗杠）
- BU_GANG（若服务器判定可补杠）
- TING（若服务器判定可报听）

**其他玩家允许操作：**

- TRUSTEE（托管）
- RECONNECT（重连）

**禁止操作：**

- PENG
- MING_GANG
- WIN（非行动玩家）

**流转条件：**

玩家出牌 → WAIT_RESPONSE

玩家暗杠 → WAIT_GANG_DRAW

玩家补杠 → WAIT_GANG_DRAW

玩家胡牌 → SETTLEMENT

---

## S-007 WAIT_RESPONSE

**说明：**

玩家出牌后，等待其他三家响应（碰或明杠）。

服务器设置响应超时时间（默认15秒）。

**允许操作（其他三家）：**

- PENG（若服务器判定可碰）
- MING_GANG（若服务器判定可明杠）
- PASS（放弃响应）

**禁止操作：**

- WIN
- TING
- DISCARD
- AN_GANG
- BU_GANG

**流转条件：**

有玩家碰牌 → WAIT_PENG_DISCARD

有玩家明杠 → WAIT_GANG_DRAW

所有玩家PASS或超时 → WAIT_DRAW（下家摸牌）

---

## S-008 WAIT_DRAW

**说明：**

等待当前行动玩家摸牌。

服务器自动执行摸牌，无需客户端请求。

**服务器执行：**

```
摸牌
  ↓
是基本花？
  ↓
是 → BU_HUA流程 → 再次摸牌 → 循环
  ↓
否 → 进入 WAIT_ACTION
```

**流转条件：**

摸牌完成（非基本花）→ WAIT_ACTION

牌墙摸完 → SETTLEMENT（流局）

---

## S-009 WAIT_ACTION

**说明：**

玩家摸牌完成，服务器计算当前所有允许操作，等待玩家决定。

**服务器计算：**

```
是否可胡？ → 是 → actions 加入 WIN
是否可暗杠？ → 是 → actions 加入 AN_GANG
是否可补杠？ → 是 → actions 加入 BU_GANG
是否可报听？ → 是 → actions 加入 TING
始终允许 → actions 加入 DISCARD
```

**允许操作（当前玩家）：**

- WIN
- AN_GANG
- BU_GANG
- TING
- DISCARD

**禁止操作：**

- PENG
- MING_GANG

**流转条件：**

玩家DISCARD → WAIT_RESPONSE

玩家WIN → SETTLEMENT

玩家AN_GANG → WAIT_GANG_DRAW

玩家BU_GANG → WAIT_GANG_DRAW

玩家TING → 更新报听状态 → 回到WAIT_ACTION（等待出牌）

---

## S-010 WAIT_PENG_DISCARD

**说明：**

玩家碰牌成功，等待该玩家打出一张牌。

**允许操作（碰牌玩家）：**

- DISCARD

**禁止操作：**

- WIN
- PENG
- GANG（任何）
- TING

**流转条件：**

玩家DISCARD → WAIT_RESPONSE

---

## S-011 WAIT_GANG_DRAW

**说明：**

玩家完成杠牌（明杠/暗杠/补杠/补花），从杠尾摸牌。

服务器自动执行摸牌。

**服务器执行：**

```
从杠尾摸牌
  ↓
是基本花？
  ↓
是 → BU_HUA流程 → 再次从杠尾摸牌 → 循环
  ↓
否 → 进入 WAIT_ACTION
      ↓
    记录本次摸牌来源为 GANG_DRAW（用于杠开判定）
```

**流转条件：**

摸牌完成（非基本花）→ WAIT_ACTION

牌墙摸完 → SETTLEMENT（流局）

---

## S-012 SETTLEMENT

**说明：**

本局结束，进行花数计算和结算。

**服务器执行：**

```
1. 确定基础胡型（Base Type）
2. 识别特殊牌型（Pattern）
3. 计算清一色加成（Modifier）
4. 计算事件奖励（Event）
5. 计算附加奖励（Bonus）
6. 汇总最终花数
7. 计算每家支付金额
8. 广播 SETTLEMENT 消息
```

流局时：

```
不结算花数
不支付金额
广播 ROUND_DRAW
```

**流转条件：**

结算完成 → ROUND_END

---

## S-013 ROUND_END

**说明：**

本局结束，等待开始下一局。

**服务器执行：**

```
判断庄家是否变更：
  胡牌玩家是庄家 → 庄家不变
  流局 → 庄家轮换到下家
  非庄家胡牌 → 庄家轮换到下家

判断是否继续：
  总局数未达到设定将数 → 准备下一局
  已达到将数 → ROOM_CLOSED
```

**允许操作：**

- NEXT_ROUND（房主）
- ROOM_DISMISS

**流转条件：**

继续下一局 → WAIT_READY

将数结束 → ROOM_CLOSED

---

## S-014 ROOM_CLOSED

**说明：**

房间已关闭，所有玩家断开连接。

不允许任何操作。

---

# 五、补花子流程（BU_HUA Sub-Flow）

补花在以下状态中均可触发：

- INITIAL_BU_HUA（开局补花）
- WAIT_DRAW（普通摸牌后）
- WAIT_GANG_DRAW（杠牌后）

补花流程：

```
摸到基本花
  ↓
基本花加入花牌区
  ↓
从杠尾补一张牌
  ↓
是基本花？
  ↓
是 → 循环
  ↓
否 → 
  ↓
检查是否可胡（包括天胡判定）
  ↓
检查是否可暗杠
  ↓
检查是否可补杠
  ↓
进入 WAIT_ACTION
```

---

# 六、必须杠判定流程

当上家打出一张牌，下家拥有暗刻时：

```
检查下家是否有对应暗刻
  ↓
有 →
  ↓
计算杠后听牌范围
  ↓
与杠前听牌范围对比
  ↓
范围不变或更大 → 必须杠（服务器强制执行）
  ↓
范围缩小 → 允许玩家自由选择
```

注意：此规则仅适用于上家打出的牌，其余家打出的牌不强制。

---

# 七、报听状态限制表

| 操作 | 报听前 | 报听后 |
|------|--------|--------|
| DISCARD | ✅ | ✅（仅限不改变听牌） |
| PENG | ✅ | ❌ |
| MING_GANG | ✅ | ✅（杠后仍听牌） |
| BU_GANG | ✅ | ✅（杠后仍听牌） |
| AN_GANG | ✅ | ✅（杠后仍听牌） |
| TING | ✅ | ❌（已报听） |
| WIN | ✅ | ✅ |
| PASS（放弃胡牌） | ✅ | ✅ |

---

# 八、状态超时处理

| 状态 | 超时时间 | 超时行为 |
|------|----------|----------|
| WAIT_DISCARD | 15秒 | 自动打出摸到的牌 |
| WAIT_RESPONSE | 10秒 | 自动PASS |
| WAIT_ACTION | 15秒 | 自动打出最后摸到的牌 |
| WAIT_PENG_DISCARD | 15秒 | 自动打出摸到的牌 |

超时触发托管逻辑。

---

# 九、断线重连状态恢复

玩家断线后：

1. 服务器保留玩家座位
2. 该玩家进入托管状态
3. 服务器继续游戏

玩家重连后：

1. 服务器验证 roomId + 昵称
2. 恢复玩家原座位
3. 推送当前完整游戏状态（RECONNECT_STATE）
4. 取消托管状态

重连状态数据包含：

```
当前状态（gameState）
手牌（handTiles）
花牌（flowers）
碰杠区（melds）
弃牌区（discards）
剩余牌数（remainTiles）
是否报听（isTing）
当前行动玩家（currentSeat）
```

---

# 十、状态机与消息协议对应关系

| 消息 | 触发状态 | 流转到 |
|------|----------|--------|
| ROOM_JOIN | IDLE / WAIT_JOIN | WAIT_JOIN / WAIT_READY |
| PLAYER_READY | WAIT_READY | DEALING（4人全准备） |
| DISCARD | WAIT_DISCARD / WAIT_ACTION / WAIT_PENG_DISCARD | WAIT_RESPONSE |
| PENG | WAIT_RESPONSE | WAIT_PENG_DISCARD |
| MING_GANG | WAIT_RESPONSE | WAIT_GANG_DRAW |
| AN_GANG | WAIT_ACTION | WAIT_GANG_DRAW |
| BU_GANG | WAIT_ACTION | WAIT_GANG_DRAW |
| TING | WAIT_ACTION | WAIT_ACTION（更新状态） |
| WIN | WAIT_ACTION / WAIT_DISCARD | SETTLEMENT |
| PASS | WAIT_RESPONSE / WAIT_ACTION | 继续当前流程 |
| RECONNECT | 任意 | 恢复当前状态 |
| TRUSTEE | 任意 | 保持当前状态 |
| ROOM_DISMISS | WAIT_JOIN 至 ROUND_END | ROOM_CLOSED |

---

# 十一、维护规范

1. 任何新增操作必须先更新本文档。
2. 任何状态流转变更必须先更新本文档。
3. Rule Cases（04）必须覆盖所有状态流转路径。
4. 服务器代码中状态名称必须与本文档完全一致。

