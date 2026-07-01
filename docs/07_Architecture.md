# 天长麻将微信小程序（Tianchang Mahjong Mini Program）

# 07_Architecture

Version：1.0.0

Status：Draft

Author：Shawn Hsu & Claude

---

# 一、文档目的

本文档定义天长麻将项目的完整技术架构。

包括：

- 技术选型
- 系统架构
- 目录结构
- 模块职责
- 编码规范

所有开发必须严格遵守本文档。

---

# 二、技术选型

## 2.1 客户端

| 技术 | 版本 | 说明 |
|------|------|------|
| 微信原生小程序 | 最新稳定版 | 前端框架 |
| JavaScript | ES6+ | 编程语言 |
| WXML | — | 页面模板 |
| WXSS | — | 页面样式 |

不使用：

- Vue / React / Taro
- 任何第三方 UI 组件库

原因：

减少审核风险，降低依赖。

---

## 2.2 服务端

| 技术 | 版本 | 说明 |
|------|------|------|
| Node.js | 18+ LTS | 运行环境 |
| TypeScript | 5.x | 编程语言 |
| ws | 最新稳定版 | WebSocket 库 |
| Express | 4.x | HTTP 服务（仅用于健康检查） |
| MySQL | 8.x | 数据库 |

---

## 2.3 部署

| 组件 | 方案 |
|------|------|
| 服务器 | 腾讯云 CVM |
| 数据库 | 腾讯云 MySQL |
| 域名 | 自备（HTTPS必须） |
| 进程管理 | PM2 |

---

# 三、系统架构图

```
┌──────────────────────────────────────────────────────────┐
│                     微信客户端                            │
│                                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ 玩家 A  │  │ 玩家 B  │  │ 玩家 C  │  │ 玩家 D  │   │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘   │
└───────┼─────────────┼─────────────┼─────────────┼───────┘
        │             │             │             │
        └─────────────┴──── WSS ────┴─────────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Node.js 服务器   │
                    │                   │
                    │  ┌─────────────┐  │
                    │  │ WebSocket   │  │
                    │  │  Handler    │  │
                    │  └──────┬──────┘  │
                    │         │         │
                    │  ┌──────▼──────┐  │
                    │  │   Router    │  │
                    │  └──────┬──────┘  │
                    │         │         │
                    │  ┌──────▼──────┐  │
                    │  │  Game Core  │  │
                    │  │             │  │
                    │  │ ┌─────────┐ │  │
                    │  │ │  Room   │ │  │
                    │  │ │ Manager │ │  │
                    │  │ └─────────┘ │  │
                    │  │ ┌─────────┐ │  │
                    │  │ │  State  │ │  │
                    │  │ │ Machine │ │  │
                    │  │ └─────────┘ │  │
                    │  │ ┌─────────┐ │  │
                    │  │ │  Rule   │ │  │
                    │  │ │ Engine  │ │  │
                    │  │ └─────────┘ │  │
                    │  │ ┌─────────┐ │  │
                    │  │ │ Score   │ │  │
                    │  │ │ Engine  │ │  │
                    │  │ └─────────┘ │  │
                    │  └─────────────┘  │
                    │         │         │
                    │  ┌──────▼──────┐  │
                    │  │    MySQL    │  │
                    │  └─────────────┘  │
                    └───────────────────┘
```

---

# 四、项目目录结构

```
tianchang_majiang/
│
├── docs/                          # 项目文档（本目录）
│   ├── 00_Project_Charter.md
│   ├── 01_Product_PRD.md
│   ├── 02_Game_Rules.md
│   ├── 03_Scoring_Table.md
│   ├── 04_Rule_Cases.md
│   ├── 05_Message_Protocol.md
│   ├── 06_State_Machine.md
│   ├── 07_Architecture.md
│   ├── 08_Database.md
│   ├── INDEX.md
│   └── README.md
│
├── client/                        # 微信小程序客户端
│   ├── app.js                     # 全局入口
│   ├── app.json                   # 全局配置
│   ├── app.wxss                   # 全局样式
│   ├── pages/
│   │   ├── index/                 # 首页（登录/创建/加入）
│   │   │   ├── index.js
│   │   │   ├── index.json
│   │   │   ├── index.wxml
│   │   │   └── index.wxss
│   │   ├── room/                  # 房间等待页
│   │   │   ├── room.js
│   │   │   ├── room.json
│   │   │   ├── room.wxml
│   │   │   └── room.wxss
│   │   ├── game/                  # 游戏主页面
│   │   │   ├── game.js
│   │   │   ├── game.json
│   │   │   ├── game.wxml
│   │   │   └── game.wxss
│   │   └── settlement/            # 结算页面
│   │       ├── settlement.js
│   │       ├── settlement.json
│   │       ├── settlement.wxml
│   │       └── settlement.wxss
│   └── utils/
│       ├── websocket.js           # WebSocket 封装
│       ├── constants.js           # 常量定义
│       └── tile.js                # 牌面显示工具
│
├── server/                        # Node.js 服务端
│   ├── src/
│   │   ├── index.ts               # 服务器入口
│   │   ├── websocket/
│   │   │   ├── handler.ts         # WebSocket 消息处理
│   │   │   └── heartbeat.ts       # 心跳管理
│   │   ├── room/
│   │   │   ├── RoomManager.ts     # 房间管理
│   │   │   └── Room.ts            # 房间实体
│   │   ├── game/
│   │   │   ├── GameEngine.ts      # 游戏引擎（核心调度）
│   │   │   ├── StateMachine.ts    # 状态机
│   │   │   ├── Deck.ts            # 牌库（洗牌/发牌）
│   │   │   ├── Player.ts          # 玩家实体
│   │   │   └── GameState.ts       # 游戏状态数据结构
│   │   ├── rules/
│   │   │   ├── WinChecker.ts      # 胡牌合法性判断
│   │   │   ├── HandAnalyzer.ts    # 手牌分析（胡型识别）
│   │   │   ├── TingChecker.ts     # 听牌判断
│   │   │   ├── GangChecker.ts     # 杠牌判断
│   │   │   └── PengChecker.ts     # 碰牌判断
│   │   ├── scoring/
│   │   │   ├── ScoreEngine.ts     # 花数计算入口
│   │   │   ├── BaseType.ts        # 基础胡型计算
│   │   │   ├── Pattern.ts         # 特殊牌型计算
│   │   │   ├── Modifier.ts        # 清一色加成
│   │   │   ├── Event.ts           # 事件奖励（天胡/杠开/海底）
│   │   │   └── Bonus.ts           # 附加奖励（大绝/暗绝）
│   │   ├── db/
│   │   │   ├── connection.ts      # 数据库连接
│   │   │   ├── GameRecord.ts      # 对局记录
│   │   │   └── Settlement.ts      # 结算记录
│   │   └── types/
│   │       ├── Tile.ts            # 牌的类型定义
│   │       ├── Message.ts         # 消息类型定义
│   │       └── GameTypes.ts       # 游戏状态类型定义
│   ├── test/
│   │   ├── rules/                 # 规则单元测试
│   │   ├── scoring/               # 计分单元测试
│   │   └── integration/           # 集成测试
│   ├── package.json
│   ├── tsconfig.json
│   └── .env                       # 环境变量（不提交Git）
│
├── test/                          # 跨端测试工具
│   └── simulate/
│       └── test-join.js           # 模拟玩家加入脚本
│
├── .gitignore
└── README.md
```

---

# 五、模块职责

## 5.1 客户端模块

### websocket.js

负责：

- 建立 WebSocket 连接
- 消息发送封装
- 断线重连逻辑
- 心跳维持

不负责：

- 任何游戏逻辑判断
- 花数计算
- 状态管理

### game.js（游戏页面）

负责：

- 接收服务器广播
- 更新 data（界面数据）
- 用户点击事件 → 发送消息到服务器
- 动画播放

不负责：

- 判断是否可碰/可杠/可胡
- 花数计算
- 任何规则判定

---

## 5.2 服务端模块

### GameEngine.ts（核心调度）

职责：

- 接收 WebSocket 消息
- 调用 StateMachine 验证当前状态
- 分发到对应处理模块
- 广播结果给所有玩家

### StateMachine.ts（状态机）

职责：

- 维护当前游戏状态
- 验证消息在当前状态是否合法
- 执行状态流转
- 不包含任何业务逻辑

### RuleEngine 模块（rules/）

职责：

- WinChecker：判断手牌是否可胡
- HandAnalyzer：分析胡牌牌型（胡型识别）
- TingChecker：判断当前是否可报听，计算所有听牌张
- GangChecker：判断是否可杠，杠后是否仍听牌
- PengChecker：判断是否可碰

每个 Checker 必须：

- 只输入手牌数据，只输出判断结果
- 不修改任何游戏状态
- 必须有对应单元测试

### ScoreEngine 模块（scoring/）

职责：

- 严格按照 03_Scoring_Table.md 计算花数
- 按顺序：BaseType → Pattern → Modifier → Event → Bonus
- 输出完整计分明细

ScoreEngine 不得：

- 修改任何游戏状态
- 直接操作数据库
- 包含任何规则判定逻辑

---

# 六、牌的数据结构

## 6.1 单张牌（Tile）

```typescript
interface Tile {
  suit: 'w' | 't' | 'b' | 'z';   // 万 / 条 / 饼 / 字牌（基本花）
  num: number | string;            // 1-9（数牌）或 'east'/'south'/'west'/'north'/'zhong'/'fa'/'bai'（字牌）
  id: string;                      // 唯一ID，例如 "w1_0"（第一张一万）
}
```

## 6.2 牌组（Meld）

```typescript
interface Meld {
  type: 'peng' | 'ming_gang' | 'an_gang' | 'bu_gang';
  tiles: Tile[];
  fromSeat?: number;               // 碰/明杠时，来自哪家打出的牌
}
```

## 6.3 玩家状态（PlayerState）

```typescript
interface PlayerState {
  seat: number;                    // 座位（0=庄家，1=南，2=西，3=北）
  nickname: string;
  handTiles: Tile[];               // 手牌（其他玩家不可见）
  flowers: Tile[];                 // 花牌区（公开）
  melds: Meld[];                   // 碰杠区（公开）
  isTing: boolean;                 // 是否已报听
  isTrustee: boolean;              // 是否托管
  isConnected: boolean;            // 是否在线
}
```

## 6.4 游戏状态（GameState）

```typescript
interface GameState {
  gameId: string;
  roomId: string;
  state: GameStateEnum;            // 当前状态机状态
  dealer: number;                  // 庄家座位
  currentSeat: number;             // 当前行动玩家
  wall: Tile[];                    // 牌墙（服务器私有）
  discards: Array<{
    tile: Tile;
    seat: number;
  }>;                              // 公开弃牌区
  players: PlayerState[];
  lastDraw: {
    tile: Tile;
    source: 'wall' | 'gang';       // 摸牌来源（普通/杠尾）
  } | null;
  isInitialPhase: boolean;         // 是否处于开局补花阶段
  round: number;                   // 当前局数
  totalRounds: number;             // 总局数（将数×4）
}
```

---

# 七、编码规范

## 7.1 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类 | PascalCase | `GameEngine` |
| 函数 | camelCase | `checkWin` |
| 常量 | UPPER_SNAKE | `MAX_PLAYERS` |
| 状态枚举 | UPPER_SNAKE | `WAIT_DISCARD` |
| 消息类型 | UPPER_SNAKE | `PLAYER_DISCARD` |
| 文件名（服务端） | PascalCase | `GameEngine.ts` |
| 文件名（客户端） | camelCase | `websocket.js` |

## 7.2 模块原则

- 每个模块只做一件事（单一职责）
- 规则判定模块不修改游戏状态
- 计分模块不包含规则判定
- 所有花数来自 03_Scoring_Table.md，不得硬编码

## 7.3 错误处理

- 所有 WebSocket 消息必须有 try-catch
- 非法消息必须返回明确错误码
- 服务器错误不得直接暴露给客户端
- 所有错误必须写入日志

## 7.4 测试要求

- rules/ 下所有 Checker 必须有单元测试
- scoring/ 下所有计算必须有单元测试
- 测试用例必须覆盖 04_Rule_Cases.md 中所有 P0 案例

---

# 八、安全规范

- 所有游戏逻辑在服务器执行，客户端零信任
- 手牌数据只发给对应玩家，不广播
- 暗杠只广播操作，不广播牌面
- 听牌信息不广播给其他玩家
- messageId 防重放攻击
- WebSocket 必须使用 wss://（生产环境）

---

# 九、维护规范

1. 任何架构变更必须先更新本文档。
2. 新增模块必须在目录结构中体现。
3. 模块职责变更必须同步更新第五章。
4. 数据结构变更必须同步更新第六章。

