```dataview
LIST
FROM "03_Areas" OR "04_Resources"
WHERE project = this.file.link
AND status != "沉淀"
SORT file.mtime DESC
```
![](assets/AI一人公司/file-20260110075529488.png)