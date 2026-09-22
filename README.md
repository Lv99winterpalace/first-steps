# 网页版扫雷游戏 — SDD 规范文档

> **文档版本**：v1.0  
> **创建日期**：2026-09-19  
> **开发方式**：SDD（Specification-Driven Development，规范驱动开发）  
> **技术栈**：Next.js 16 + TypeScript + Tailwind CSS 4 + shadcn/ui

---

## 目录

1. [项目概述](#1-项目概述)
2. [术语表](#2-术语表)
3. [游戏规则](#3-游戏规则)
4. [功能需求](#4-功能需求)
5. [非功能需求](#5-非功能需求)
6. [UI/UX 设计规范](#6-uiux-设计规范)
7. [技术架构](#7-技术架构)
8. [数据结构定义](#8-数据结构定义)
9. [核心算法](#9-核心算法)
10. [组件设计](#10-组件设计)
11. [开发阶段与依赖关系](#11-开发阶段与依赖关系)
12. [验收标准](#12-验收标准)

---

## 1. 项目概述

### 1.1 项目目标

开发一个网页版本的扫雷游戏，采用现代扁平化设计风格，支持多难度、深浅主题切换、计时计分、本地排行榜、音效反馈与胜利/失败动画。游戏在浏览器中直接运行，无需安装。

### 1.2 目标用户

- 喜欢经典扫雷玩法的休闲玩家
- 希望在浏览器中快速进行一局游戏的用户
- 对现代视觉风格有偏好的用户

### 1.3 核心价值

- **经典玩法**：完整还原 Windows 扫雷的核心规则与操作
- **现代体验**：扁平化设计、流畅动画、主题切换
- **即开即玩**：纯前端实现，加载即玩，数据本地持久化

---

## 2. 术语表

| 术语 | 含义 |
|------|------|
| Cell | 雷区中的一个格子 |
| Mine | 地雷 |
| Flag | 旗帜标记，表示"我认为这里有雷" |
| Question | 问号标记，表示"不确定" |
| Reveal | 揭开格子 |
| Chord | 和弦展开，双击数字格时自动展开周围未标记格子 |
| Flood Fill | 连锁展开，揭开 0 格时自动展开相邻空白区域 |
| First-click Safety | 首点保护，第一次点击保证不踩雷 |
| HUD | Heads-Up Display，顶部信息显示区（计时器、雷数等） |

---

## 3. 游戏规则

### 3.1 基本规则

1. 雷区是一个 `rows × cols` 的网格，其中随机分布 `mineCount` 颗地雷。
2. 玩家通过点击格子来揭开它们：
   - 若揭开的是地雷，游戏失败。
   - 若揭开的是数字格，显示该格周围 8 格中的地雷数量。
   - 若揭开的是空白格（周围 0 颗雷），自动连锁揭开相邻的空白格与边界数字格（Flood Fill）。
3. 玩家可以右键给格子插旗（标记为"有雷"），再次右键切换为问号，第三次右键取消标记。
4. 当所有非雷格子都被揭开时，游戏胜利。

### 3.2 首点保护

- 第一次左键点击的格子及其周围 8 格保证不会是地雷。
- 实现方式：在第一次点击后才生成雷区，生成时排除首点及其邻居。

### 3.3 和弦展开（Chord Click）

- 双击（或同时左右键）一个已揭开的数字格：
  - 若该数字格周围已插旗的格子数等于该数字，则自动揭开周围所有**未插旗**的格子。
  - 若插旗位置错误（旗子标在了非雷格上），则会揭开雷，游戏失败。
- 这是高级玩家加速通关的核心操作。

### 3.4 难度等级

| 难度 | 行数 | 列数 | 雷数 |
|------|------|------|------|
| 初级 Beginner | 9 | 9 | 10 |
| 中级 Intermediate | 16 | 16 | 40 |
| 高级 Expert | 16 | 30 | 99 |
| 自定义 Custom | 用户设定（行 5–24，列 5–30） | 用户设定 | 用户设定（≤ 行×列−1） |

---

## 4. 功能需求

### 4.1 核心玩法（P0 必须）

| 编号 | 功能 | 描述 |
|------|------|------|
| F-01 | 雷区渲染 | 根据 rows×cols 渲染网格 |
| F-02 | 左键揭开 | 点击未揭开格，执行揭开逻辑 |
| F-03 | Flood Fill | 揭开 0 格时连锁展开 |
| F-04 | 右键标记 | 右键循环切换：无 → 旗 → 问号 → 无 |
| F-05 | 双击和弦 | 双击数字格执行和弦展开 |
| F-06 | 首点保护 | 第一次点击不踩雷 |
| F-07 | 胜负判定 | 踩雷失败 / 全部非雷格揭开则胜利 |
| F-08 | 重新开始 | 一键重置当前难度的新局 |

### 4.2 难度系统（P0）

| 编号 | 功能 | 描述 |
|------|------|------|
| F-09 | 难度切换 | 初级/中级/高级一键切换 |
| F-10 | 自定义难度 | 用户输入行、列、雷数，校验合法性 |

### 4.3 计分系统（P0）

| 编号 | 功能 | 描述 |
|------|------|------|
| F-11 | 计时器 | 从第一次有效点击开始计时，胜利/失败时停止 |
| F-12 | 雷数计数器 | 显示"剩余雷数 = 总雷数 − 已插旗数" |
| F-13 | 本地排行榜 | 按难度记录前 10 名最佳时间（localStorage） |
| F-14 | 胜利录入 | 胜利时若进入前 10，提示输入名字并保存 |

### 4.4 主题（P1）

| 编号 | 功能 | 描述 |
|------|------|------|
| F-15 | 深色/浅色切换 | 顶部按钮切换主题，记忆用户选择 |
| F-16 | 主题记忆 | 通过 localStorage 记住用户上次选择 |

### 4.5 音效（P1）

| 编号 | 功能 | 描述 |
|------|------|------|
| F-17 | 揭开音效 | 左键揭开格子时播放短促"click" |
| F-18 | 爆炸音效 | 踩雷时播放爆炸音 |
| F-19 | 胜利音效 | 胜利时播放胜利旋律 |
| F-20 | 标记音效 | 插旗/取消标记时播放短音 |
| F-21 | 音效开关 | 顶部按钮可静音/取消静音，记忆用户选择 |

### 4.6 动画与反馈（P1）

| 编号 | 功能 | 描述 |
|------|------|------|
| F-22 | 格子揭开动画 | 揭开时有轻微缩放/淡入动画 |
| F-23 | 爆炸动画 | 踩雷格子有红色闪烁，所有雷格依次揭开 |
| F-24 | 胜利动画 | 所有非雷格变绿，撒花/彩带效果 |
| F-25 | 表情按钮反馈 | 顶部表情按钮：正常😊 / 紧张😮（按下时）/ 胜利😎 / 失败😵 |

---

## 5. 非功能需求

| 编号 | 维度 | 要求 |
|------|------|------|
| N-01 | 性能 | 高级难度（16×30=480 格）渲染 < 100ms，Flood Fill < 50ms |
| N-02 | 兼容性 | 支持 Chrome / Edge / Firefox / Safari 最新版 |
| N-03 | 响应式 | 桌面端为主，窗口缩放时雷区居中且不溢出 |
| N-04 | 可访问性 | 键盘可操作（Tab 切换格子、空格揭开、F 插旗） |
| N-05 | 数据持久化 | 排行榜与主题/音效偏好通过 localStorage 保存 |
| N-06 | 代码质量 | TypeScript 严格模式，核心逻辑单元测试覆盖 |
| N-07 | 无后端 | 纯前端实现，无需服务端 |

---

## 6. UI/UX 设计规范

### 6.1 整体布局

```
┌─────────────────────────────────────────┐
│  顶部工具栏                              │
│  [😊 重新开始]  [💣 010]  [⏱ 000]  [⚙]  │
├─────────────────────────────────────────┤
│  难度选择栏                              │
│  [初级] [中级] [高级] [自定义]            │
├─────────────────────────────────────────┤
│                                         │
│         雷区（居中显示）                  │
│                                         │
│   ┌─┬─┬─┬─┬─┬─┬─┬─┬─┐                  │
│   │ │ │ │ │ │ │ │ │ │                  │
│   ├─┼─┼─┼─┼─┼─┼─┼─┼─┤                  │
│   │ │ │ │ │ │ │ │ │ │                  │
│   └─┴─┴─┴─┴─┴─┴─┴─┴─┘                  │
│                                         │
├─────────────────────────────────────────┤
│  底部：[🏆 排行榜]  [🌙 主题]  [🔊 音效] │
└─────────────────────────────────────────┘
```

### 6.2 配色方案（现代扁平化）

**浅色主题：**
- 背景：`#F8FAFC`（slate-50）
- 雷区背景：`#E2E8F0`（slate-200）
- 未揭开格：`#94A3B8`（slate-400）渐变到 `#64748B`（slate-500）
- 已揭开格：`#F1F5F9`（slate-100）
- 数字配色（1-8）：蓝、绿、红、深蓝、棕、青、黑、灰
- 强调色：`#3B82F6`（blue-500）

**深色主题：**
- 背景：`#0F172A`（slate-900）
- 雷区背景：`#1E293B`（slate-800）
- 未揭开格：`#475569`（slate-600）渐变到 `#334155`（slate-700）
- 已揭开格：`#1E293B`（slate-800）
- 数字配色：相应提亮
- 强调色：`#60A5FA`（blue-400）

### 6.3 格子状态视觉

| 状态 | 浅色 | 深色 | 内容 |
|------|------|------|------|
| 未揭开 | slate-400 渐变 | slate-600 渐变 | 空 |
| 已揭开-空白 | slate-100 | slate-800 | 空 |
| 已揭开-数字 | slate-100 | slate-800 | 彩色数字 |
| 已揭开-雷 | red-500 | red-600 | 💣 |
| 插旗 | slate-400 | slate-600 | 🚩 |
| 问号 | slate-400 | slate-600 | ❓ |
| 踩雷（爆炸点） | red-600 闪烁 | red-700 闪烁 | 💥 |
| 错误插旗（失败时） | slate-400 + 红叉 | slate-600 + 红叉 | 🚩❌ |

### 6.4 动画规范

- 格子揭开：`transform: scale(0.95) → 1`，`opacity: 0 → 1`，时长 150ms
- 格子悬停（未揭开）：`brightness(1.1)`，过渡 100ms
- 爆炸动画：背景色 red-500 → red-700 闪烁 3 次，时长 600ms
- 胜利动画：所有非雷格背景变绿（emerald-400），撒花粒子 2s
- 表情按钮：按下时切换为 😮，松开恢复

---

## 7. 技术架构

### 7.1 技术栈

| 层 | 技术 | 用途 |
|----|------|------|
| 框架 | Next.js 16 (App Router) | 应用骨架 |
| 语言 | TypeScript (strict) | 类型安全 |
| 样式 | Tailwind CSS 4 | 原子化 CSS |
| 组件库 | shadcn/ui | 基础组件（Button、Dialog 等） |
| 状态管理 | React useState + useReducer + Context | 局部状态 + 全局主题 |
| 持久化 | localStorage | 排行榜、主题、音效偏好 |
| 音效 | Web Audio API | 程序化生成音效（无需音频文件） |
| 动画 | CSS Transition + Framer Motion | 轻量动画 |
| 测试 | Vitest | 核心逻辑单元测试 |

### 7.2 目录结构

```
minesweeper/
├── src/
│   ├── app/
│   │   ├── page.tsx              # 主页面
│   │   ├── layout.tsx            # 根布局（主题 Provider）
│   │   └── globals.css           # 全局样式
│   ├── components/
│   │   ├── game/
│   │   │   ├── Board.tsx         # 雷区容器
│   │   │   ├── Cell.tsx          # 单个格子
│   │   │   ├── HUD.tsx           # 顶部信息栏
│   │   │   ├── DifficultySelector.tsx  # 难度选择
│   │   │   ├── GameOverlay.tsx   # 胜利/失败遮罩
│   │   │   └── EmojiButton.tsx   # 表情按钮
│   │   ├── leaderboard/
│   │   │   ├── LeaderboardDialog.tsx  # 排行榜弹窗
│   │   │   └── NameInputDialog.tsx   # 录入名字
│   │   └── ui/                   # shadcn/ui 组件
│   ├── game/
│   │   ├── types.ts              # 类型定义
│   │   ├── logic.ts              # 核心游戏逻辑（纯函数）
│   │   ├── generator.ts          # 雷区生成
│   │   ├── reveal.ts             # 揭开/展开算法
│   │   └── __tests__/
│   │       └── logic.test.ts    # 单元测试
│   ├── hooks/
│   │   ├── useGame.ts            # 游戏状态 Hook
│   │   ├── useTimer.ts           # 计时器 Hook
│   │   ├── useTheme.ts          # 主题 Hook
│   │   ├── useSound.ts          # 音效 Hook
│   │   └── useLeaderboard.ts    # 排行榜 Hook
│   ├── lib/
│   │   ├── storage.ts           # localStorage 封装
│   │   └── utils.ts             # 工具函数
│   └── styles/
│       └── theme.css             # 主题变量
├── public/
├── package.json
└── tsconfig.json
```

### 7.3 状态管理策略

- **游戏状态**：使用 `useReducer` 集中管理（board、gameStatus、mineCount、flagCount、time、difficulty）
- **主题**：使用 React Context 全局提供，配合 `next-themes` 或自实现
- **音效偏好**：与主题一起放在 Context
- **排行榜**：通过 `useLeaderboard` Hook 封装 localStorage 读写

---

## 8. 数据结构定义

### 8.1 格子状态枚举

```typescript
// src/game/types.ts

export enum CellState {
  Hidden = 0,     // 未揭开
  Revealed = 1,   // 已揭开
  Flagged = 2,    // 插旗
  Question = 3,   // 问号
}

export enum GameStatus {
  Ready = 'ready',       // 未开始（等待首点）
  Playing = 'playing',   // 进行中
  Won = 'won',           // 胜利
  Lost = 'lost',         // 失败
}

export type Difficulty = 'beginner' | 'intermediate' | 'expert' | 'custom';

export interface Cell {
  row: number;
  col: number;
  isMine: boolean;           // 是否是雷
  adjacentMines: number;     // 周围雷数（0-8）
  state: CellState;          // 当前状态
  isExploded: boolean;       // 是否是踩中的那颗雷（爆炸点）
}

export interface BoardConfig {
  rows: number;
  cols: number;
  mineCount: number;
}

export interface GameState {
  board: Cell[][];            // 二维网格
  config: BoardConfig;
  status: GameStatus;
  flagCount: number;          // 已插旗数
  revealedCount: number;      // 已揭开数
  time: number;               // 已用秒数
  difficulty: Difficulty;
  firstClickDone: boolean;    // 是否已首点
}

export interface LeaderboardEntry {
  name: string;
  time: number;       // 秒
  date: string;       // ISO 日期
  difficulty: Difficulty;
}

export interface Leaderboard {
  beginner: LeaderboardEntry[];
  intermediate: LeaderboardEntry[];
  expert: LeaderboardEntry[];
}
```

### 8.2 难度预设

```typescript
export const DIFFICULTY_PRESETS: Record<Exclude<Difficulty, 'custom'>, BoardConfig> = {
  beginner:     { rows: 9,  cols: 9,  mineCount: 10 },
  intermediate: { rows: 16, cols: 16, mineCount: 40 },
  expert:       { rows: 16, cols: 30, mineCount: 99 },
};

export const CUSTOM_LIMITS = {
  rows: { min: 5, max: 24 },
  cols: { min: 5, max: 30 },
  mineCount: { min: 1, max: (rows, cols) => rows * cols - 1 },
};
```

---

## 9. 核心算法

### 9.1 雷区生成（首点保护）

```typescript
// src/game/generator.ts

function generateBoard(config: BoardConfig, safeCell: { row: number; col: number }): Cell[][] {
  const { rows, cols, mineCount } = config;
  const board: Cell[][] = createEmptyBoard(rows, cols);

  // 计算禁区：首点 + 周围 8 格
  const forbidden = new Set<string>();
  for (let dr = -1; dr <= 1; dr++) {
    for (let dc = -1; dc <= 1; dc++) {
      const r = safeCell.row + dr;
      const c = safeCell.col + dc;
      if (r >= 0 && r < rows && c >= 0 && c < cols) {
        forbidden.add(`${r},${c}`);
      }
    }
  }

  // 在禁区外随机放雷
  const candidates: string[] = [];
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (!forbidden.has(`${r},${c}`)) candidates.push(`${r},${c}`);
    }
  }
  shuffle(candidates);
  for (let i = 0; i < mineCount; i++) {
    const [r, c] = candidates[i].split(',').map(Number);
    board[r][c].isMine = true;
  }

  // 计算每格的 adjacentMines
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (!board[r][c].isMine) {
        board[r][c].adjacentMines = countAdjacentMines(board, r, c);
      }
    }
  }

  return board;
}
```

### 9.2 Flood Fill 揭开算法（BFS）

```typescript
// src/game/reveal.ts

function revealCell(board: Cell[][], row: number, col: number): { board: Cell[][]; revealed: Cell[] } {
  const rows = board.length;
  const cols = board[0].length;
  const newBoard = cloneBoard(board);
  const revealed: Cell[] = [];

  const queue: [number, number][] = [[row, col]];
  const visited = new Set<string>([`${row},${col}`]);

  while (queue.length > 0) {
    const [r, c] = queue.shift()!;
    const cell = newBoard[r][c];

    // 跳过已揭开/已插旗/问号
    if (cell.state !== CellState.Hidden) continue;

    cell.state = CellState.Revealed;
    revealed.push(cell);

    // 若是 0 格，继续展开邻居
    if (!cell.isMine && cell.adjacentMines === 0) {
      for (let dr = -1; dr <= 1; dr++) {
        for (let dc = -1; dc <= 1; dc++) {
          if (dr === 0 && dc === 0) continue;
          const nr = r + dr;
          const nc = c + dc;
          if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
            const key = `${nr},${nc}`;
            if (!visited.has(key)) {
              visited.add(key);
              queue.push([nr, nc]);
            }
          }
        }
      }
    }
  }

  return { board: newBoard, revealed };
}
```

### 9.3 和弦展开（Chord）

```typescript
function chordCell(board: Cell[][], row: number, col: number): { board: Cell[][]; hitMine: boolean } {
  const cell = board[row][col];
  if (cell.state !== CellState.Revealed || cell.adjacentMines === 0) {
    return { board, hitMine: false };
  }

  // 统计周围插旗数
  const neighbors = getNeighbors(board, row, col);
  const flagCount = neighbors.filter(n => n.state === CellState.Flagged).length;

  if (flagCount !== cell.adjacentMines) {
    return { board, hitMine: false };
  }

  // 揭开周围所有未插旗的格子
  let newBoard = cloneBoard(board);
  let hitMine = false;
  for (const n of neighbors) {
    if (n.state === CellState.Hidden) {
      if (n.isMine) {
        hitMine = true;
        newBoard[n.row][n.col].isExploded = true;
        newBoard[n.row][n.col].state = CellState.Revealed;
      } else {
        const result = revealCell(newBoard, n.row, n.col);
        newBoard = result.board;
      }
    }
  }

  return { board: newBoard, hitMine };
}
```

### 9.4 胜负判定

```typescript
function checkWin(state: GameState): boolean {
  const totalCells = state.config.rows * state.config.cols;
  const nonMineCells = totalCells - state.config.mineCount;
  return state.revealedCount === nonMineCells;
}

function checkLose(cell: Cell): boolean {
  return cell.isMine && cell.isExploded;
}
```

### 9.5 右键标记循环

```typescript
function cycleFlag(cell: Cell): CellState {
  switch (cell.state) {
    case CellState.Hidden:    return CellState.Flagged;
    case CellState.Flagged:   return CellState.Question;
    case CellState.Question:  return CellState.Hidden;
    default:                  return cell.state; // 已揭开的不变
  }
}
```

---

## 10. 组件设计

### 10.1 组件树

```
<App>
  <ThemeProvider>
    <GameProvider>  (可选，或直接在 page 内用 useReducer)
      <Page>
        <Header>
          <EmojiButton />       # 重新开始 + 表情
          <MineCounter />       # 雷数计数器
          <Timer />             # 计时器
          <SettingsButton />    # 设置（音效、主题）
        </Header>
        <DifficultySelector />
        <Board>
          {cells.map(cell => <Cell />)}
        </Board>
        <Footer>
          <LeaderboardButton />
          <ThemeToggle />
          <SoundToggle />
        </Footer>
        <GameOverlay />         # 胜利/失败遮罩（条件渲染）
        <LeaderboardDialog />   # 排行榜弹窗
        <NameInputDialog />     # 录入名字弹窗
      </Page>
    </GameProvider>
  </ThemeProvider>
</App>
```

### 10.2 关键组件职责

| 组件 | 职责 | 主要 Props |
|------|------|-----------|
| `Board` | 渲染网格，处理点击事件委托 | `board`, `onCellClick`, `onCellRightClick`, `onCellChord` |
| `Cell` | 渲染单个格子，根据状态显示内容 | `cell`, `onClick`, `onContextMenu`, `onDoubleClick` |
| `HUD` | 顶部信息栏（表情、雷数、计时） | `status`, `mineCount`, `flagCount`, `time`, `onRestart` |
| `DifficultySelector` | 难度切换按钮组 | `current`, `onChange` |
| `GameOverlay` | 胜利/失败遮罩 + 动画 | `status`, `time`, `onRestart`, `onViewLeaderboard` |
| `LeaderboardDialog` | 排行榜表格 | `entries`, `difficulty` |
| `NameInputDialog` | 胜利录入名字 | `onSubmit`, `onSkip` |

### 10.3 关键 Hook

| Hook | 职责 |
|------|------|
| `useGame` | 封装 useReducer，暴露 state + actions（reveal、flag、chord、restart、changeDifficulty） |
| `useTimer` | 计时器：开始/停止/重置，每秒 +1 |
| `useTheme` | 主题切换 + 持久化 |
| `useSound` | 音效播放 + 静音状态 + 持久化 |
| `useLeaderboard` | 排行榜 CRUD + 持久化 |

---

## 11. 开发阶段与依赖关系

### 11.1 依赖关系图

```
[Phase A: 项目脚手架]
    │
    ▼
[Phase B: 核心逻辑层]  ← 无 UI 依赖，可独立测试
    │
    ├──→ [Phase C: 基础 UI 组件] (Cell, Board)
    │       │
    │       └──→ [Phase D: 交互接入] (左键/右键/双击)
    │               │
    │               └──→ [Phase E: HUD 与难度] (计时器、雷数、难度切换)
    │                       │
    │                       ├──→ [Phase F: 主题切换]
    │                       ├──→ [Phase G: 音效系统]
    │                       ├──→ [Phase H: 排行榜]
    │                       └──→ [Phase I: 动画与遮罩]
    │                               │
    │                               ▼
    └─────────────────────────→ [Phase J: 联调与打磨]
```

### 11.2 阶段任务清单

#### Phase A — 项目脚手架
- A1. 使用 fullstack-dev skill 初始化 Next.js 项目
- A2. 配置 Tailwind CSS 4
- A3. 安装 shadcn/ui 基础组件（Button、Dialog、Input）
- A4. 创建目录结构骨架

#### Phase B — 核心逻辑层（纯函数，可单测）
- B1. 定义类型 `types.ts`
- B2. 实现雷区生成 `generator.ts`（含首点保护）
- B3. 实现 Flood Fill 揭开 `reveal.ts`
- B4. 实现和弦展开 `chord.ts`
- B5. 实现右键标记循环
- B6. 实现胜负判定
- B7. 编写单元测试 `logic.test.ts`

#### Phase C — 基础 UI 组件
- C1. `Cell` 组件（根据状态渲染不同样式）
- C2. `Board` 组件（网格布局 + 事件委托）

#### Phase D — 交互接入
- D1. `useGame` Hook（useReducer 整合所有逻辑）
- D2. 左键揭开接入
- D3. 右键标记接入
- D4. 双击和弦接入
- D5. 首点保护触发雷区生成

#### Phase E — HUD 与难度
- E1. `HUD` 组件（表情按钮、雷数计数器、计时器）
- E2. `useTimer` Hook
- E3. `DifficultySelector` 组件
- E4. 自定义难度输入与校验
- E5. 重新开始功能

#### Phase F — 主题切换
- F1. `useTheme` Hook + ThemeProvider
- F2. CSS 变量定义（浅色/深色）
- F3. 主题切换按钮
- F4. 主题持久化

#### Phase G — 音效系统
- G1. `useSound` Hook（Web Audio API 程序化音效）
- G2. 揭开、爆炸、胜利、标记音效实现
- G3. 音效开关按钮 + 持久化

#### Phase H — 排行榜
- H1. `storage.ts` localStorage 封装
- H2. `useLeaderboard` Hook
- H3. `LeaderboardDialog` 组件
- H4. `NameInputDialog` 胜利录入
- H5. 胜利时自动判断是否进入前 10

#### Phase I — 动画与遮罩
- I1. 格子揭开动画（CSS transition）
- I2. 爆炸动画（踩雷闪烁 + 全雷揭开）
- I3. 胜利动画（撒花/彩带）
- I4. `GameOverlay` 胜利/失败遮罩
- I5. 表情按钮状态联动

#### Phase J — 联调与打磨
- J1. 整体布局与响应式调整
- J2. 键盘可访问性
- J3. 边界情况测试（最小/最大难度、自定义极端值）
- J4. 性能验证（高级难度渲染）
- J5. 最终视觉打磨

---

## 12. 验收标准

### 12.1 功能验收

| 编号 | 验收项 | 通过标准 |
|------|--------|---------|
| V-01 | 初级难度可正常完成一局 | 胜利时显示遮罩与时间 |
| V-02 | 中级、高级难度雷数与网格正确 | 16×16=40 雷，16×30=99 雷 |
| V-03 | 自定义难度校验 | 行 5-24、列 5-30、雷 ≤ 行×列−1 |
| V-04 | 首点保护 | 第一次点击及其 8 邻居均非雷 |
| V-05 | Flood Fill | 揭开 0 格时连锁展开至数字边界 |
| V-06 | 右键标记循环 | 无→旗→问号→无 |
| V-07 | 和弦展开 | 双击数字格，旗数匹配时揭开周围 |
| V-08 | 错误和弦失败 | 旗标错位置时和弦会踩雷 |
| V-09 | 计时器 | 首点开始，胜负停止 |
| V-10 | 雷数计数器 | 总雷数 − 已插旗数 |
| V-11 | 排行榜 | 各难度前 10 名，胜利时录入 |
| V-12 | 主题切换 | 浅色/深色即时切换并记忆 |
| V-13 | 音效 | 揭开/爆炸/胜利/标记音效正常 |
| V-14 | 音效开关 | 静音后无声，记忆偏好 |
| V-15 | 胜利动画 | 非雷格变绿 + 撒花 |
| V-16 | 失败动画 | 踩雷格闪烁 + 全雷揭开 |
| V-17 | 表情按钮 | 正常/紧张/胜利/失败四态 |

### 12.2 非功能验收

| 编号 | 验收项 | 通过标准 |
|------|--------|---------|
| NF-01 | 性能 | 高级难度渲染 < 100ms |
| NF-02 | 兼容性 | Chrome/Edge/Firefox/Safari 正常 |
| NF-03 | 持久化 | 刷新后排行榜、主题、音效偏好保留 |
| NF-04 | 类型安全 | `tsc --noEmit` 无错误 |
| NF-05 | 单元测试 | 核心逻辑测试通过 |

---

## 附录 A：音效设计（Web Audio API 程序化生成）

| 音效 | 波形 | 频率 | 时长 | 描述 |
|------|------|------|------|------|
| 揭开 | square | 800Hz → 1200Hz | 50ms | 短促 click |
| 标记 | triangle | 600Hz | 80ms | 轻柔 |
| 爆炸 | sawtooth + noise | 200Hz → 50Hz | 500ms | 低沉爆炸 |
| 胜利 | sine | C-E-G-C 上行 | 800ms | 胜利旋律 |

## 附录 B：localStorage 键名

| 键 | 值 | 用途 |
|----|----|----|
| `minesweeper:leaderboard` | JSON `Leaderboard` | 排行榜 |
| `minesweeper:theme` | `'light' \| 'dark'` | 主题偏好 |
| `minesweeper:sound` | `'on' \| 'off'` | 音效偏好 |
| `minesweeper:difficulty` | `Difficulty` | 上次难度 |
