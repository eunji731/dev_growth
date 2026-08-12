---
cssclasses:
  - escape-dashboard
---

# 🏠 ESCAPE HQ

> 실행은 **TickTick**, 기록과 지식은 **Obsidian**

`BUTTON[daily-open, weekly-open]`
```meta-bind-button
label: 🔥 오늘 기록 열기
style: primary
id: daily-open
hidden: true

actions:
  - type: command
    command: daily-notes
```
```meta-bind-button
label: 📅 이번 주 열기
icon: ""
style: primary
class: ""
cssStyle: ""
backgroundImage: ""
tooltip: ""
id: weekly-open
hidden: true
actions:
  - type: command
    command: periodic-notes:open-weekly-note

```

## 🎯 CURRENT FOCUS

```dataviewjs
const weeks = dv.pages('"05 Weekly"')
    .where(p => p.type === "weekly")
    .sort(p => p.file.name, "desc")
    .array();

if (!weeks.length) {
    dv.paragraph("이번 주 기록이 없습니다.");
} else {
    const w = weeks[0];

    const root = document.createElement("div");
    root.className = "weekly-board";
    dv.container.appendChild(root);

    // 현재 집중 영역
    const focusPanel = document.createElement("div");
    focusPanel.className = "weekly-focus";

    const focusTitle = document.createElement("div");
    focusTitle.className = "weekly-eyebrow";
    focusTitle.textContent = "CURRENT FOCUS";
    focusPanel.appendChild(focusTitle);

    const focusGrid = document.createElement("div");
    focusGrid.className = "weekly-focus-grid";

    const focuses = w.focus
        ? (Array.isArray(w.focus) ? w.focus : [w.focus])
        : [];

    for (const item of focuses) {
        const [name = "", status = "", current = ""] =
            String(item).split("|").map(v => v.trim());

        const card = document.createElement("div");
        card.className = "weekly-focus-card";

        card.innerHTML = `
            <div class="weekly-focus-name">${name}</div>
            <div class="weekly-focus-current">${current}</div>
            <div class="weekly-focus-status">${status}</div>
        `;

        focusGrid.appendChild(card);
    }

    focusPanel.appendChild(focusGrid);
    root.appendChild(focusPanel);

    // 아래 3영역
    const bottom = document.createElement("div");
    bottom.className = "weekly-bottom";

    function makeList(title, icon, values, cls) {
        const panel = document.createElement("div");
        panel.className = `weekly-list-panel ${cls}`;

        const heading = document.createElement("div");
        heading.className = "weekly-list-title";
        heading.textContent = `${icon} ${title}`;
        panel.appendChild(heading);

        const list = document.createElement("ul");

        const items = values
            ? (Array.isArray(values) ? values : [values])
            : [];

        if (!items.length) {
            const li = document.createElement("li");
            li.textContent = "아직 없음";
            list.appendChild(li);
        }

        for (const value of items) {
            const li = document.createElement("li");
            li.textContent = value;
            list.appendChild(li);
        }

        panel.appendChild(list);
        bottom.appendChild(panel);
    }

    makeList("이번 주 목표", "🎯", w.goals, "weekly-goals");
    makeList("이번 주 성과", "🏆", w.outputs, "weekly-output");
    makeList("막힌 것", "🚧", w.blockers, "weekly-blocker");

    root.appendChild(bottom);
}
```
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

## 🗂 RECENT

```dataviewjs
const daily = dv.pages('"01 Daily"')
    .sort(p => p.file.name, 'desc')
    .limit(5)
    .array();

const study = dv.pages('"02 Study"')
    .where(p => !p.file.name.toLowerCase().startsWith("test"))
    .sort(p => p.file.mtime, 'desc')
    .limit(8)
    .array();

const root = document.createElement("div"); root.className = "recent-grid"; dv.container.appendChild(root);

function makePanel(title, pages, type) {
    const panel = document.createElement("div");
    panel.className = "recent-panel";

    const heading = document.createElement("div");
    heading.className = "recent-title";
    heading.textContent = title;
    panel.appendChild(heading);

    const list = document.createElement("div");
    list.className = "recent-list";
    panel.appendChild(list);

    if (!pages.length) {
        const empty = document.createElement("div");
        empty.className = "recent-empty";
        empty.textContent = "아직 기록이 없습니다.";
        list.appendChild(empty);
    }

    for (const p of pages) {
        const item = document.createElement("a");
        item.className = "recent-item internal-link";
        item.setAttribute("data-href", p.file.path);
        item.href = p.file.path;

        if (type === "daily") {
            const area = p.study_area
                ? (Array.isArray(p.study_area) ? p.study_area.join(" · ") : p.study_area)
                : "기록";

            const minutes = Number(p.study_minutes) || 0;

            item.innerHTML = `
                <div class="recent-item-title">${p.file.name}</div>
                <div class="recent-item-sub">${area}${minutes ? ` · ${minutes}분` : ""}</div>
            `;
        } else {
            const area = p.area || p.topic || "Study";

            item.innerHTML = `
                <div class="recent-item-title">${p.file.name}</div>
                <div class="recent-item-sub">${area}</div>
            `;
        }

        list.appendChild(item);
    }

    root.appendChild(panel);
}

makePanel("📝 RECENT DAILY", daily, "daily");
makePanel("🧠 RECENT STUDY", study, "study");
```

## 🚀 QUICK ACCESS

> [!links]
> [[02 Study/Infra/Network|🌐 Network]]
> [[02 Study/Infra/Docker|🐳 Docker]]
> [[02 Study/Infra/Kubernetes|☸️ Kubernetes]]
> [[02 Study/Linux/Linux|🐧 Linux]]
>
> [[02 Study/React/React|⚛️ React]]
> [[02 Study/TypeScript/TypeScript|🔷 TypeScript]]
> [[02 Study/Node/Node|🟢 Node.js]]
> [[02 Study/CS/CS|📖 CS]]
>
> [[04 Career/경력기술서|💼 Career]]
> [[03 Work/업무 기록|🗂 Work]]
> [[00 Inbox/Inbox|📥 Inbox]]

