建议 MVP 先采用 **前端本地状态 +** **TypeScript** **类型 + mockData**，暂时不接数据库。核心数据结构先保留 4 个主对象：

```Plain
Player
Team
Match
Event
```

---

# 1.Player 玩家数据结构
    

```TypeScript
export type Stage = "dreamer" | "challenger" | "professional" | "ending";

export type Position = "TOP" | "JUG" | "MID" | "ADC" | "SUP";

export interface Player {
  id: string;
  name: string;
  age: number;
  year: number;
  month: number;

  stage: Stage;
  position: Position;
  teamId?: string;

  // 五维底层属性
  talent: number;
  effort: number;
  mental: number;
  social: number;
  ambition: number;

  // 状态属性
  form: number;
  fatigue: number;
  injury: number;

  // 梦想者/挑战者关键进度
  dreamValue: number;
  careerAttention: number;
  coachRating: number;
  starterScore: number;

  // 职业阶段成长
  popularity: number;
  honor: number;
  influence: number;
  legendScore: number;

  // 统计
  matchesPlayed: number;
  wins: number;
  losses: number;
  mvpCount: number;
  championships: number;

  // 标签
  tags: string[];
}
```

## MVP说明

`stage` 决定当前人生阶段。 `year/month` 驱动时间推进。 `careerAttention` 用于梦想者阶段进入挑战者。 `starterScore` 用于挑战者阶段进入职业者。 `popularity/honor/legendScore` 用于职业者阶段总结。

---

# 2.Team 战队数据结构
    

```TypeScript
export type TeamLevel = "internet_cafe" | "youth" | "ldl" | "lpl";

export interface TeamMember {
  id: string;
  name: string;
  position: Position;
  rating: number;
  personalityTag: string;
  relationship: number;
}

export interface Coach {
  id: string;
  name: string;
  style: "strict" | "mentor" | "tactical" | "loose";
  tactic: number;
  development: number;
  management: number;
  relationship: number;
}

export interface Team {
  id: string;
  name: string;
  level: TeamLevel;

  rating: number;
  chemistry: number;

  members: TeamMember[];
  coach?: Coach;

  wins: number;
  losses: number;
  rank?: number;

  tags: string[];
}
```

## MVP说明

梦想者阶段的 Team 可以是“网吧队”。

挑战者阶段是“青训队 / LDL队”。

职业者阶段是“LPL战队”。

同一个页面展示不同阶段的组织即可，不需要做多套 UI。

---

# 3.Match 比赛数据结构
    

```TypeScript
export type MatchType =
  | "rank"
  | "internet_cafe"
  | "training"
  | "ldl"
  | "lpl_regular"
  | "lpl_playoff"
  | "final";

export interface Match {
  id: string;
  year: number;
  month: number;

  type: MatchType;
  title: string;

  playerTeamId?: string;
  opponentTeamName: string;

  playerScore: number; // 0-100
  teamScore: number;
  opponentScore: number;

  result: "win" | "loss";
  scoreText: string; // "2:0", "2:1", "3:2"

  kda: string;
  isMvp: boolean;
  isFinalMvp: boolean;

  rewards: {
    popularity?: number;
    honor?: number;
    careerAttention?: number;
    coachRating?: number;
    starterScore?: number;
  };

  summary: string;
}
```

## MVP说明

比赛不作为行动。

普通比赛在月末自动结算。

关键比赛用弹窗展示结果。

---

# 4.Event 事件数据结构
    

```TypeScript
export type EventStage = "dreamer" | "challenger" | "professional";

export type EventCategory =
  | "growth"
  | "setback"
  | "relationship"
  | "dream"
  | "coach"
  | "team"
  | "media";

export interface EventOption {
  id: string;
  text: string;

  effects: Partial<{
    talent: number;
    effort: number;
    mental: number;
    social: number;
    ambition: number;

    form: number;
    fatigue: number;
    dreamValue: number;
    careerAttention: number;
    coachRating: number;
    starterScore: number;
    popularity: number;
    honor: number;
  }>;

  nextEventId?: string;
  addTag?: string;
}

export interface GameEvent {
  id: string;
  stage: EventStage;
  category: EventCategory;

  title: string;
  description: string;

  trigger: Partial<{
    minAge: number;
    minDreamValue: number;
    minCareerAttention: number;
    minCoachRating: number;
    minStarterScore: number;
    minPopularity: number;
    requiredTag: string;
  }>;

  options: EventOption[];

  weight: number;
  once: boolean;
}
```

## MVP说明

事件系统先做最小版本：

```Plain
根据当前阶段
↓
筛选满足 trigger 的事件
↓
按 weight 随机抽取
↓
玩家选择
↓
修改 Player 数值
```

---

# 5.GameState 总状态结构
    

实际开发中建议再包一层：

```TypeScript
export interface GameState {
  player: Player;
  teams: Team[];
  matches: Match[];
  eventsSeen: string[];

  currentEvent?: GameEvent;
  currentMatchResults: Match[];

  currentPage: "home" | "matches" | "relationships" | "team" | "profile";
}
```

---

# 6.MVP 数据流
    

```Plain
玩家点击“结束本月”
↓
结算本月行动
↓
更新 Player 属性
↓
抽取 Event
↓
玩家选择事件选项
↓
自动结算本月比赛
↓
检查阶段晋升
↓
year/month +1
```

---

# 7.阶段晋升规则
    

```TypeScript
function checkStageTransition(player: Player): Stage {
  if (player.stage === "dreamer" && player.careerAttention >= 100) {
    return "challenger";
  }

  if (player.stage === "challenger" && player.starterScore >= 100) {
    return "professional";
  }

  return player.stage;
}
```

---

# 8.MVP 最小 mock 数据文件建议
    

```Plain
/src
  /types
    game.ts
  /data
    initialPlayer.ts
    teams.ts
    events.ts
  /lib
    gameLogic.ts
    matchLogic.ts
    eventLogic.ts
```

---

这套数据结构已经足够支撑：

```Plain
创建角色
↓
月度行动
↓
事件选择
↓
比赛结算
↓
梦想者 → 挑战者 → 职业者
↓
Demo结束
```