### 📥 收件箱 (当前有 `$= dv.pages('"01 - Inbox"').length` 个待处理)
```dataview
LIST FROM "01 - Inbox" LIMIT 5
```
## 进行中的项目

```dataview
TABLE status AS "状态", file.mtime AS "更新于"
FROM "01 - Projects"
WHERE status = "运行中"
SORT file.mtime DESC
```
## 最近加入的资源
```dataview
LIST FROM "04 - Resources"
SORT file.ctime DESC
LIMIT 5
```
## 本周待办

```tasks
not done
due before next monday
group by priority
```

