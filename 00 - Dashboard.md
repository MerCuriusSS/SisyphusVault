---
cssclasses: cards, cards-1-1, cards-cols-3
---

# 🚀 个人调度中心

> [!multi-column]
>
> > ### 📥 1. 收件箱 (Inbox)
> > ```dataview
> > LIST FROM "Inbox" LIMIT 5
> > ```
>
> > ### ⛏️ 2. 知识矿 (Resources)
> > ```dataview
> > LIST FROM "Resources" WHERE !processed LIMIT 5
> > ```
>
> > ### 🏗️ 3. 活跃项目 (Projects)
> > ```dataview
> > TABLE status AS "状态" FROM "Projects" WHERE status = "active"
> > ```

---

> [!multi-column]
>
> > ### 📂 4. 领域异常检查 (Areas)
> > ```dataview
> > LIST FROM "Areas" WHERE status != "沉淀"
> > ```
>
> > ### ✅ 5. 本周待办事项
> > ```dataview
> > TASK WHERE !completed AND (due <= date(today) + dur(7 days) OR !due)
> > ```