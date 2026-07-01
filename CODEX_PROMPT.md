# Tianchang Mahjong — Codex Development Prompt

## Project Repository

https://github.com/shawnhsu922/tianchang_majiang

## Your First Task

Read ALL documents in the /docs folder before writing a single line of code.

Required reading order:
1. README.md
2. 00_Project_Charter.md
3. 01_Product_PRD.md
4. 02_Game_Rules.md
5. 03_Scoring_Table.md
6. 04_Rule_Cases.md
7. 05_Message_Protocol.md
8. 06_State_Machine.md
9. 07_Architecture.md
10. 08_Database.md

Do not start coding until you have read all 10 documents.

---

## Project Summary

A WeChat Mini Program for 4-player real-time Tianchang Mahjong.

- Client: WeChat native Mini Program (JavaScript)
- Server: Node.js 18+ with TypeScript, WebSocket (ws library)
- Database: MySQL 8.x
- Deployment: Tencent Cloud

---

## Directory Structure

Follow 07_Architecture.md exactly:

tianchang_majiang/
├── docs/          # Do not modify
├── client/        # WeChat Mini Program
├── server/        # Node.js server
└── test/          # Test tools

---

## Development Rules

### Must follow:
1. All game logic runs on the server. Client is display-only.
2. All flower counts come from 03_Scoring_Table.md. No hardcoding.
3. All state transitions follow 06_State_Machine.md exactly.
4. All WebSocket messages follow 05_Message_Protocol.md exactly.
5. All directory and module structure follows 07_Architecture.md.
6. Every module must pass corresponding P0 cases in 04_Rule_Cases.md before moving on.

### Strictly forbidden:
- Modifying any game rules
- Modifying any flower counts
- Adding features not in 01_Product_PRD.md
- Client-side rule calculation (win check, gang check, flower count)
- Hardcoding flower numbers in code
- Skipping tests

---

## Development Phases

### Phase 1 — Server Foundation

Goal: Server starts, accepts WebSocket connections, room creation and joining works.

Steps:
1. Initialize server/ with TypeScript config → verify: `tsc` compiles without errors
2. Implement WebSocket handler (05_Message_Protocol.md) → verify: client can connect
3. Implement RoomManager (ROOM_CREATE, ROOM_JOIN) → verify: RC-ROOM cases pass
4. Implement MySQL connection and users/rooms/room_players tables (08_Database.md) → verify: tables created

---

### Phase 2 — Game Core

Goal: 4 players can start a game and receive dealt tiles.

Steps:
1. Implement Deck.ts (136 tiles, shuffle, deal) → verify: correct tile counts per player
2. Implement INITIAL_BU_HUA state (06_State_Machine.md S-005) → verify: RC-BU-HUA-001 to 007 pass
3. Implement GameState.ts data structures (07_Architecture.md section 6) → verify: state serializes correctly
4. Implement DEALING → INITIAL_BU_HUA → WAIT_DISCARD flow → verify: game reaches WAIT_DISCARD

---

### Phase 3 — Rule Engine

Goal: All core game actions work correctly.

Build in this order, verifying Rule Cases after each:

1. WinChecker.ts → verify: RC-WIN-001 to 003 pass
2. HandAnalyzer.ts (base types) → verify: RC-WIN-004 to 008 pass
3. PengChecker.ts → verify: RC-PENG-001 to 005 pass
4. GangChecker.ts → verify: RC-MING-GANG, RC-BU-GANG, RC-AN-GANG cases pass
5. TingChecker.ts → verify: RC-TING-001 to 012 pass
6. HandAnalyzer.ts (special patterns) → verify: RC-WIN-009 to 046 pass

---

### Phase 4 — Score Engine

Goal: Flower count calculation is 100% correct per 03_Scoring_Table.md.

Build in this order:
1. BaseType.ts → verify: RC-SETTLEMENT-001 to 010 pass
2. Pattern.ts → verify: RC-SETTLEMENT-011 to 029 pass
3. Modifier.ts (清一色) → verify: all 清一色 settlement cases pass
4. Event.ts (天胡/杠开/海底) → verify: RC-SETTLEMENT-030 to 033 pass
5. Bonus.ts (大绝/暗绝) → verify: RC-WIN-035 to 038 pass
6. ScoreEngine.ts (orchestrator) → verify: all REGRESSION cases pass

Critical rules for Bonus.ts:
- 单边大绝: player wins with a tile that is a 绝张 (all 4 publicly visible as peng/ming_gang), AND player has more than one winning tile. +10
- 双边大绝: player has exactly 2 winning tiles, both are 绝张. +20
- 三边大绝: player has exactly 3 winning tiles, all are 绝张. +30
- 暗绝: the winning tile completes an existing anke (player already held 3 of that tile). +10
- 天外来客 (SP-016, Pattern not Bonus): player's ONLY winning tile (ya-zi shape) is a 绝张. This is mutually exclusive with 大绝.
- 绝张 is determined ONLY by publicly visible peng and ming_gang. Hidden tiles in other players' hands do not count.

---

### Phase 5 — Settlement & Round Management

Goal: Full game loop works end to end.

Steps:
1. Implement SETTLEMENT state → verify: RC-SETTLEMENT cases pass
2. Implement player_bills and settlements DB writes (08_Database.md) → verify: records written correctly
3. Implement ROUND_END庄家 rotation logic (02_Game_Rules.md GR-004) → verify: dealer changes correctly
4. Implement 将数 tracking → verify: game ends after correct number of rounds

---

### Phase 6 — Reconnect & Trustee

Goal: Disconnected players can rejoin without breaking the game.

Steps:
1. Implement RECONNECT flow (05_Message_Protocol.md MSG-018, 06_State_Machine.md section 9) → verify: RC-TING-010, RC-TING-011 pass
2. Implement TRUSTEE auto-play (discard only, no ting/gang) → verify: game continues when player disconnects
3. Implement heartbeat (MSG-019) → verify: server detects disconnect within timeout

---

### Phase 7 — WeChat Client

Goal: Players can play a full game on their phones.

Build pages in order:
1. index (login + create/join room) → verify: room code appears after creation
2. room (waiting room, player list, jiang setting) → verify: 4 players see each other
3. game (tile display, discard, peng/gang/ting/win actions) → verify: full game playable
4. settlement (score breakdown, payment amounts) → verify: matches server SETTLEMENT message

Client rules:
- Never calculate any game logic client-side
- All actions send a message and wait for server response
- All UI state comes from server broadcasts only

---

## Success Criteria for V1.0

- [ ] 4 players complete a full game without errors
- [ ] All rule judgments are correct
- [ ] All flower counts match 03_Scoring_Table.md
- [ ] Disconnected player can rejoin mid-game
- [ ] Settlement amounts are correct
- [ ] All RC P0 test cases pass

---

## If You Are Uncertain

- Check the relevant doc first
- If docs conflict, ask — do not guess
- If a rule is ambiguous, ask — do not interpret silently
- Never simplify a rule without explicit approval
