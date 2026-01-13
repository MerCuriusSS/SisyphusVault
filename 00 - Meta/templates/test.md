<%*
// 核心：用 Templater 内置变量自动生成内容
const today = tp.date.now("YYYY-MM-DD"); // 当前日期
const time = tp.date.now("HH:mm"); // 当前时间
const noteName = tp.file.title; // 当前笔记文件名
// 渲染内容到笔记中
tR.set(`
# 【日记】${today} ${time}
## 笔记文件名：${noteName}
## 今日完成
- 
## 待办
- 
## 感想
- 
`);
%>