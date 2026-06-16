## 1.users / saves 存档表
    

```SQL
saves {
  id: string
  save_name: string
  created_at: datetime
  updated_at: datetime
  game_state: json
}
```

MVP 最简单做法：把完整 `GameState` 存成 JSON。

---

## 2.players 玩家表
    

```SQL
players {
  id: string
  save_id: string

  name: string
  age: number
  year: number
  month: number
  stage: "dreamer" | "challenger" | "professional" | "ending"
  position: "TOP" | "JUG" | "MID" | "ADC" | "SUP"
  team_id: string

  talent: number
  effort: number
  mental: number
  social: number
  ambition: number

  form: number
  fatigue: number
  injury: number

  dream_value: number
  career_attention: number
  coach_rating: number
  starter_score: number

  popularity: number
  honor: number
  influence: number
  legend_score: number

  matches_played: number
  wins: number
  losses: number
  mvp_count: number
  championships: number

  tags: json
}
```

---

## 3.teams 战队表
    

```SQL
teams {
  id: string
  save_id: string

  name: string
  level: "internet_cafe" | "youth" | "ldl" | "lpl"

  rating: number
  chemistry: number

  wins: number
  losses: number
  rank: number

  tags: json
}
```

---

## 4.team_members 队员表
    

```SQL
team_members {
  id: string
  save_id: string
  team_id: string

  name: string
  position: "TOP" | "JUG" | "MID" | "ADC" | "SUP"
  rating: number
  personality_tag: string
  relationship: number
}
```

---

## 5.coaches 教练表
    

```SQL
coaches {
  id: string
  save_id: string
  team_id: string

  name: string
  style: "strict" | "mentor" | "tactical" | "loose"

  tactic: number
  development: number
  management: number

  relationship: number
}
```

---

## 6.matches 比赛记录表
    

```SQL
matches {
  id: string
  save_id: string

  year: number
  month: number

  type: "rank" | "internet_cafe" | "training" | "ldl" | "lpl_regular" | "lpl_playoff" | "final"

  title: string
  player_team_id: string
  opponent_team_name: string

  player_score: number
  team_score: number
  opponent_score: number

  result: "win" | "loss"
  score_text: string

  kda: string
  is_mvp: boolean
  is_final_mvp: boolean

  rewards: json
  summary: text
}
```

---

## 7.events 事件配置表
    

```SQL
events {
  id: string

  stage: "dreamer" | "challenger" | "professional"
  category: string

  title: string
  description: text

  trigger: json
  options: json

  weight: number
  once: boolean
}
```

说明：`events` 是配置表，不一定跟随存档变化。

---

## 8.seen_events 已触发事件表
    

```SQL
seen_events {
  id: string
  save_id: string
  event_id: string
  triggered_at_year: number
  triggered_at_month: number
}
```

---

# MVP 最推荐实现方式

第一版不用真的建 8 张表，直接这样：

```TypeScript
interface SaveData {
  id: string;
  saveName: string;
  gameState: GameState;
  createdAt: string;
  updatedAt: string;
}
```

然后存在：

```TypeScript
localStorage.setItem("lol-life-save", JSON.stringify(saveData));
```

等 Demo 跑通后，再把这些结构迁移成数据库表。

# 最小落地优先级

```Plain
第一阶段：localStorage 保存整个 GameState
第二阶段：拆分 Player / Team / Match / Event
第三阶段：接 Supabase 或 SQLite
```

这样最适合新手团队 + Codex 开发。