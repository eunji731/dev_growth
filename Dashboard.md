# 🏠 DASHBOARD

> [!info] ESCAPE PROJECT
> 오늘 할 일은 TickTick에서 관리하고, 여기는 기록과 지식으로 들어가는 입구입니다.

---

## 📊 GROWTH

```dataviewjs
const today = dv.date("today").startOf("day");
const weekStart = today.minus({ days: today.weekday - 1 });
const weekEnd = weekStart.plus({ days: 7 });

const allPages = dv.pages('"01 Daily"')
    .where(p => p.date)
    .array();

const weekPages = allPages.filter(p => {
    const d = dv.date(p.date);
    return d &&
        d.toMillis() >= weekStart.toMillis() &&
        d.toMillis() < weekEnd.toMillis();
});

const totalMinutes = weekPages.reduce(
    (sum, p) => sum + (Number(p.study_minutes) || 0), 0
);

const studyDays = weekPages.filter(
    p => (Number(p.study_minutes) || 0) > 0
).length;

const codingDays = weekPages.filter(
    p => p.coding_test === true
).length;

const areas = [...new Set(
    weekPages.flatMap(p => {
        if (!p.study_area) return [];
        return dv.isArray(p.study_area)
            ? Array.from(p.study_area)
            : [p.study_area];
    }).map(String).filter(Boolean)
)];

const hours = Math.floor(totalMinutes / 60);
const mins = totalMinutes % 60;

const timeText =
    hours && mins ? `${hours}h ${mins}m`
    : hours ? `${hours}h`
    : `${mins}m`;

const studyMap = new Map();

for (const p of allPages) {
    const d = dv.date(p.date);
    if (!d) continue;

    studyMap.set(
        d.toISODate(),
        Number(p.study_minutes) || 0
    );
}

// 연속 공부일
let streak = 0;

for (let i = 0; i < 365; i++) {
    const d = today.minus({ days: i });
    const minutes = studyMap.get(d.toISODate()) || 0;

    if (minutes > 0) {
        streak++;
    } else if (i === 0) {
        continue;
    } else {
        break;
    }
}

// 전체 카드
const root = dv.el("div", "", { cls: "growth-box" });

const top = document.createElement("div");
top.className = "growth-top";

top.innerHTML = `
    <div>
        <div class="growth-eyebrow">THIS WEEK</div>
        <div class="growth-range">
            ${weekStart.toFormat("M/d")} — ${weekEnd.minus({days:1}).toFormat("M/d")}
        </div>
    </div>

    <div class="growth-streak">
        🔥 <strong>${streak}</strong>일 연속
    </div>
`;

root.appendChild(top);

// 통계 카드
const stats = document.createElement("div");
stats.className = "growth-stats";

const items = [
    ["⏱", "공부시간", timeText],
    ["🔥", "공부일", `${studyDays}일`],
    ["✍🏻", "코테", `${codingDays}일`],
    ["📚", "학습분야", `${areas.length}개`]
];

for (const [icon, label, value] of items) {
    const card = document.createElement("div");
    card.className = "growth-stat";

    card.innerHTML = `
        <div class="growth-label">${icon} ${label}</div>
        <div class="growth-value">${value}</div>
    `;

    stats.appendChild(card);
}

root.appendChild(stats);

// 분야
if (areas.length) {
    const areaText = document.createElement("div");
    areaText.className = "growth-areas";
    areaText.textContent = areas.join(" · ");
    root.appendChild(areaText);
}

// 잔디
const streakTitle = document.createElement("div");
streakTitle.className = "growth-heat-title";
streakTitle.textContent = "최근 35일";
root.appendChild(streakTitle);

const heatmap = document.createElement("div");
heatmap.className = "growth-heatmap-new";

for (let i = 34; i >= 0; i--) {
    const d = today.minus({ days: i });
    const minutes = studyMap.get(d.toISODate()) || 0;

    let level = 0;
    if (minutes > 0) level = 1;
    if (minutes >= 30) level = 2;
    if (minutes >= 60) level = 3;
    if (minutes >= 90) level = 4;

    const cell = document.createElement("div");
    cell.className = `growth-cell level-${level}`;
    cell.title = `${d.toFormat("yyyy-MM-dd")} · ${minutes}분`;

    heatmap.appendChild(cell);
}

root.appendChild(heatmap);
```

---

## 🔥 TODAY

```meta-bind-button
label: 🔥 오늘 기록 열기
style: primary
actions:
  - type: command
    command: daily-notes
```

---
## 📝 최근 Daily ![[Recent Daily.base]]
---
## 🧠 최근 Study ![[Recent Study.base]]

---
## 🚧 INFRA

- [[02 Study/Infra/Network|🌐 Network]]
- [[02 Study/Infra/Docker|🐳 Docker]]
- [[02 Study/Infra/Kubernetes|☸️ Kubernetes]]
- [[02 Study/Linux/Linux|🐧 Linux]]

---

## 💻 FRONTEND

- [[02 Study/React/React|⚛️ React]]
- [[02 Study/TypeScript/TypeScript|🔷 TypeScript]]
- [[02 Study/Node/Node|🟢 Node.js]]

---

## 📖 CS

- [[02 Study/CS/CS|CS Index]]

---

## ✍🏻 CODING TEST

- [[02 Study/CS/Coding Test|코테 기록]]

---

## 💼 CAREER

- [[04 Career/이력서|이력서]]
- [[04 Career/경력기술서|경력기술서]]
- [[04 Career/포트폴리오|포트폴리오]]

---

## 💼 WORK

- [[03 Work/업무 기록|업무 기록]]

---

## 📥 INBOX

- [[00 Inbox/Inbox|임시 메모]]