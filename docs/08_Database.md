# 天长麻将微信小程序（Tianchang Mahjong Mini Program）

# 08_Database

Version：1.0.0

Status：Draft

Author：Shawn Hsu & AI Assistant

---

# 一、文档目的

本文档定义天长麻将项目的数据库设计。

包括：

- 表结构定义
- 字段说明
- 索引设计
- 数据流说明

---

# 二、数据库选型

MySQL 8.x

字符集：utf8mb4

排序规则：utf8mb4_unicode_ci

---

# 三、表结构

## 3.1 用户表（users）

```sql
CREATE TABLE users (
  id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  openid        VARCHAR(64)  NOT NULL UNIQUE,   -- 微信 openid
  nickname      VARCHAR(32)  NOT NULL,           -- 微信昵称
  avatar_url    VARCHAR(256) DEFAULT '',         -- 头像 URL
  created_at    DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  last_login_at DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_openid (openid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

字段说明：

| 字段 | 说明 |
|------|------|
| id | 用户唯一 ID |
| openid | 微信用户唯一标识（不公开） |
| nickname | 游戏中显示的昵称 |
| avatar_url | 头像地址 |
| created_at | 注册时间 |
| last_login_at | 最近登录时间 |

---

## 3.2 房间表（rooms）

```sql
CREATE TABLE rooms (
  id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  room_code     VARCHAR(8)   NOT NULL UNIQUE,    -- 6位房间号（玩家输入）
  host_user_id  BIGINT UNSIGNED NOT NULL,         -- 房主 user_id
  total_jiang   TINYINT UNSIGNED NOT NULL DEFAULT 2,  -- 打几将
  unit_price    SMALLINT UNSIGNED NOT NULL DEFAULT 5, -- 每张花金额（元）
  status        ENUM('waiting','playing','finished','closed') NOT NULL DEFAULT 'waiting',
  created_at    DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  closed_at     DATETIME     DEFAULT NULL,
  INDEX idx_room_code (room_code),
  INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

字段说明：

| 字段 | 说明 |
|------|------|
| room_code | 玩家输入的6位房间号 |
| host_user_id | 房主用户ID |
| total_jiang | 打几将（1/2/4） |
| unit_price | 每张花金额，默认5元 |
| status | 房间状态 |

---

## 3.3 房间玩家表（room_players）

```sql
CREATE TABLE room_players (
  id          BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  room_id     BIGINT UNSIGNED NOT NULL,
  user_id     BIGINT UNSIGNED NOT NULL,
  seat        TINYINT UNSIGNED NOT NULL,          -- 座位（0=庄家初始位,1,2,3）
  joined_at   DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uk_room_user (room_id, user_id),
  UNIQUE KEY uk_room_seat (room_id, seat),
  INDEX idx_room_id (room_id),
  INDEX idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 3.4 对局表（games）

```sql
CREATE TABLE games (
  id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  game_id       VARCHAR(32)  NOT NULL UNIQUE,     -- 对局唯一ID（如 game_000001）
  room_id       BIGINT UNSIGNED NOT NULL,
  round         TINYINT UNSIGNED NOT NULL,         -- 第几局（圈内第几牌）
  jiang         TINYINT UNSIGNED NOT NULL,         -- 第几将
  dealer_seat   TINYINT UNSIGNED NOT NULL,         -- 庄家座位
  status        ENUM('playing','finished','draw') NOT NULL DEFAULT 'playing',
  winner_seat   TINYINT UNSIGNED DEFAULT NULL,     -- 胡牌玩家座位（流局为NULL）
  started_at    DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  finished_at   DATETIME DEFAULT NULL,
  INDEX idx_room_id (room_id),
  INDEX idx_game_id (game_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

字段说明：

| 字段 | 说明 |
|------|------|
| game_id | 对局唯一标识，用于结算和回放 |
| round | 本将内第几局（1-4） |
| jiang | 第几将 |
| dealer_seat | 庄家座位编号 |
| status | 对局状态（进行中/结束/流局） |
| winner_seat | 胡牌玩家，流局为 NULL |

---

## 3.5 结算表（settlements）

```sql
CREATE TABLE settlements (
  id              BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  game_id         VARCHAR(32)  NOT NULL,
  room_id         BIGINT UNSIGNED NOT NULL,
  winner_user_id  BIGINT UNSIGNED NOT NULL,        -- 胡牌玩家 user_id

  -- 胡牌详情
  base_type       VARCHAR(32)  NOT NULL,           -- 基础胡型
  patterns        JSON         DEFAULT NULL,        -- 特殊牌型数组
  modifiers       JSON         DEFAULT NULL,        -- 加成项数组
  events          JSON         DEFAULT NULL,        -- 事件数组
  bonus           JSON         DEFAULT NULL,        -- 奖励项数组

  -- 花数与金额
  flower_count    SMALLINT UNSIGNED NOT NULL,       -- 最终花数
  unit_price      SMALLINT UNSIGNED NOT NULL,       -- 每张花金额
  pay_per_player  INT UNSIGNED NOT NULL,            -- 每家支付金额
  total_win       INT UNSIGNED NOT NULL,            -- 胡牌玩家总收入

  created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_game_id (game_id),
  INDEX idx_winner (winner_user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

字段说明：

| 字段 | 说明 |
|------|------|
| base_type | 如 "单吊" / "对倒" |
| patterns | 如 ["豪华七对", "清一色"] |
| events | 如 ["海底捞月", "杠开"] |
| bonus | 如 ["暗绝"] |
| flower_count | 最终花数 |
| pay_per_player | 每名输家支付金额 |
| total_win | 胡牌玩家总收入（pay_per_player × 3） |

---

## 3.6 玩家账单表（player_bills）

```sql
CREATE TABLE player_bills (
  id          BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  game_id     VARCHAR(32)  NOT NULL,
  room_id     BIGINT UNSIGNED NOT NULL,
  user_id     BIGINT UNSIGNED NOT NULL,
  seat        TINYINT UNSIGNED NOT NULL,
  amount      INT NOT NULL,                        -- 正数=收入，负数=支出
  created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_game_id (game_id),
  INDEX idx_user_id (user_id),
  INDEX idx_room_id (room_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

说明：

每局结算后，为每位玩家写入一条账单记录。

胡牌玩家：amount = total_win（正数）

其余三家：amount = -pay_per_player（负数）

流局：amount = 0

---

# 四、数据流说明

## 4.1 创建房间流程

```
玩家创建房间
  ↓
INSERT rooms（生成 room_code）
  ↓
INSERT room_players（seat=0，房主）
  ↓
返回 room_code 给客户端
```

---

## 4.2 加入房间流程

```
玩家输入 room_code 加入
  ↓
SELECT rooms WHERE room_code = ? AND status = 'waiting'
  ↓
INSERT room_players（分配 seat）
  ↓
广播房间状态
```

---

## 4.3 游戏开始流程

```
4人全部准备
  ↓
INSERT games（生成 game_id）
  ↓
UPDATE rooms SET status = 'playing'
  ↓
开始游戏
```

---

## 4.4 结算流程

```
玩家胡牌
  ↓
ScoreEngine 计算花数
  ↓
INSERT settlements（完整结算明细）
  ↓
INSERT player_bills × 4（每位玩家账单）
  ↓
UPDATE games SET status = 'finished', winner_seat = ?
  ↓
广播 SETTLEMENT 消息
```

---

## 4.5 流局流程

```
牌摸完无人胡
  ↓
UPDATE games SET status = 'draw'
  ↓
INSERT player_bills × 4（amount = 0）
  ↓
广播 ROUND_DRAW
```

---

# 五、注意事项

1. games 表和 settlements 表以 game_id 关联，不使用外键（性能考虑）。
2. 手牌数据不入库（仅存在服务器内存中），对局结束后销毁。
3. player_bills 的 amount 用正负数区分收支，便于后续账单汇总。
4. V1.0 不实现回放功能，牌谱数据暂不存储。
5. 所有金额单位为元（整数），不使用小数。

