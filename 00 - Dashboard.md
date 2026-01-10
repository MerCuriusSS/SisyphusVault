## 资料库

```dataview
LIST
FROM "04 - Resources"
LIMIT 10
```

## 项目处理情况

```dataview
TABLE status AS "状态", deadline AS "截止日期"
FROM "02 - Projects"
WHERE status = "active" OR status = "进行中"
```


## Inbox数量
```dataview
LIST
FROM "01 - Inbox"
LIMIT 10
```

## Area区域笔记

```dataview
TABLE status AS "当前状态", area_type AS "分类"
FROM "03 - Areas"
WHERE status != "沉淀" 
```


## 当周任务待办事项

```tasks
TASK
WHERE !completed 
AND (due <= date(today) + dur(7 days) OR !due)
WHERE !contains(section, "Template") -- 排除模板里的待办
```

