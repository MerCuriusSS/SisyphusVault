### 项目总进度
- [x] 前端项目初始化（环境准备、整合依赖、通用基础代码） ✅ 2026-06-17
- [x] 后端项目初始化（环境准备、工程化配置、组件库、通用布局、路由、请求） ✅ 2026-06-17
- [x] 用户模块 ✅ 2026-06-17
- [x] 通过AI生成应用（langchain4j、SSE流式输出） ✅ 2026-06-17
- [x] 应用模块 ✅ 2026-06-17
- [x] 对话历史模块 ✅ 2026-06-17
- [x] 工程项目生成 ✅ 2026-06-17
- [x] 生成应用封面图、下载项目代码包、AI智能选择方案 ✅ 2026-07-02
- [x] 用户可视化编辑 ✅ 2026-07-02
- [ ] AI工作流
- [x] 部署上线 ✅ 2026-07-02
- [x] 系统可观测性构建 ✅ 2026-07-02
- [ ] 微服务改造

### 项目复盘


- 技术架构
- SpringBoot 特性总结 [AI 零代码应用——SpringBoot 特性](../03%20-%20Areas/AI%20零代码应用——SpringBoot%20特性.md)
- 管理模块总结 [AI 零代码应用——管理模块](../03%20-%20Areas/AI%20零代码应用——管理模块.md)
- AI 应用生成模块总结
- 可观测性总结


#### 技术架构

🔴前端：

- 环境：
	- Node v24.14.1（≈JDK）
	- npm包管理工具（≈Maven仓库）
- 框架：
	- Vue3（≈Springboot）
	- Ant Design Vue（≈Hutool等工具包）
	- Pinia（≈本地缓存）
	- Axios
- 工程化：
	- Vite构建工具（≈Maven）
	- TypeScript（JS升级版；≈Java）
	- Eslint（代码格式检查器）

🔴后端：

- 环境：Java 21、Maven、
- 框架：SpringBoot 3.5、Mybatis-Flex
- 数据存储：MySQL、Redis、COS对象存储
- AI技术：DeepSeek LLM、LangChain4J、ToolCalling、Guardrails、RAG（内存存储+BGE-small 向量模型）
- 工具：HuTool、Lombok、knife4j、Redisson、Selenium
- 可观测性：prometheus、grafana

#### SpringBoot 特性总结

 [AI 零代码应用——SpringBoot 特性](../03%20-%20Areas/AI%20零代码应用——SpringBoot%20特性.md)

#### 管理模块总结 

[AI 零代码应用——管理模块](../03%20-%20Areas/AI%20零代码应用——管理模块.md)


#### AI 应用生成模块总结

[AI 零代码应用——AI 应用生成模块总结](../03%20-%20Areas/AI%20零代码应用——AI%20应用生成模块总结.md)

#### 可观测性

🔴核心目标：监控LLM的调用以及消费情况

🔴维度：

| 维度      | 值                                 |
| ------- | --------------------------------- |
| 模型名称    | deepseek-v4-flash/deepseek-v4-pro |
| 模型请求结果  | 成功/失败                             |
| token类型 | 输入/输出                             |
| token用量 |                                   |
| 工具调用    | 类型、次数                             |

🔴指标体系

- 各模型的请求总数
- 各模型的输入/输出token数量、花费
- 总token花费
- 工具调用次数
- 不同用户的token消费排行榜

🔴告警规则

- 紧急类通知
	- 错误率>10%
	- AI服务不可用
	- 并发数超过20
- 警告类通知
	- 当前花费>300
	- 工具调用失败率>10%
