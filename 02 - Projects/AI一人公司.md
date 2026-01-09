```dataview
LIST
FROM "03_Areas" OR "04_Resources"
WHERE project = this.file.link
AND status != "沉淀"
SORT file.mtime DESC
```
