---
tags:
  - AI/应用
  - 编程/框架
category: 技术
status: 加工
project:
application:
source:
---
### 应用创建

DB 记录
- `应用名称`
- `应用封面`
- `初始化prompt`
- `生成类型（简单/Vue）`
- `部署标识`
- `审计字段`

### AI对话

[AI零代码应用——AI对话请求](AI零代码应用——AI对话请求.md)

### 应用部署

- 校验（参数、应用信息、应用所有用户归属、生成文件路径）
- 判断文件生成类型（简单模式还是Vue工程）
- Vue工程类型调用`Hutool命令执行工具`完成构建过程
- 复制文件到部署目录
- 完成访问URL到部署目录文件的映射
- 异步生成截图并更新应用封面


```java
public String deployApp(Long appId, User loginUser) {  
    // 1. 参数校验  
    ThrowUtils.throwIf(appId == null || appId <= 0, ErrorCode.PARAMS_ERROR, "应用 ID 不能为空");  
    ThrowUtils.throwIf(loginUser == null, ErrorCode.NOT_LOGIN_ERROR, "用户未登录");  
    // 2. 查询应用信息  
    App app = this.getById(appId);  
    ThrowUtils.throwIf(app == null, ErrorCode.NOT_FOUND_ERROR, "应用不存在");  
    // 3. 验证用户是否有权限部署该应用，仅本人可以部署  
    if (!app.getUserId().equals(loginUser.getId())) {  
        throw new BusinessException(ErrorCode.NO_AUTH_ERROR, "无权限部署该应用");  
    }  
    // 提前提取 codeGenType，供指标埋点使用  
    String codeGenType = app.getCodeGenType();  
    long startNanos = System.nanoTime();  
    try {  
    // 4. 检查是否已有 deployKey    String deployKey = app.getDeployKey();  
    // 没有则生成 6 位 deployKey（大小写字母 + 数字）  
    if (StrUtil.isBlank(deployKey)) {  
        deployKey = RandomUtil.randomString(6);  
    }  
    // 5. 构建源目录路径  
    String sourceDirName = codeGenType + "_" + appId;  
    String sourceDirPath = AppConstant.CODE_OUTPUT_ROOT_DIR + File.separator + sourceDirName;  
    // 6. 检查源目录是否存在  
    File sourceDir = new File(sourceDirPath);  
    if (!sourceDir.exists() || !sourceDir.isDirectory()) {  
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "应用代码不存在，请先生成代码");  
    }  
  
    // 7. Vue 项目特殊处理：执行构建  
    CodeGenTypeEnum codeGenTypeEnum = CodeGenTypeEnum.getEnumByValue(codeGenType);  
    if (codeGenTypeEnum == CodeGenTypeEnum.VUE_PROJECT) {  
        // Vue 项目需要构建  
        boolean buildSuccess = vueProjectBuilder.buildProject(sourceDirPath);  
        ThrowUtils.throwIf(!buildSuccess, ErrorCode.SYSTEM_ERROR, "Vue 项目构建失败，请检查代码和依赖");  
        // 检查 dist 目录是否存在  
        File distDir = new File(sourceDirPath, "dist");  
        ThrowUtils.throwIf(!distDir.exists(), ErrorCode.SYSTEM_ERROR, "Vue 项目构建完成但未生成 dist 目录");  
        // 将 dist 目录作为部署源  
        sourceDir = distDir;  
        log.info("Vue 项目构建成功，将部署 dist 目录: {}", distDir.getAbsolutePath());  
    }  
  
    // 8. 复制文件到部署目录  
    String deployDirPath = AppConstant.CODE_DEPLOY_ROOT_DIR + File.separator + deployKey;  
    try {  
        FileUtil.copyContent(sourceDir, new File(deployDirPath), true);  
    } catch (Exception e) {  
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "部署失败：" + e.getMessage());  
    }  
    // 9. 更新应用的 deployKey 和部署时间  
    App updateApp = new App();  
    updateApp.setId(appId);  
    updateApp.setDeployKey(deployKey);  
    updateApp.setDeployedTime(LocalDateTime.now());  
    boolean updateResult = this.updateById(updateApp);  
    ThrowUtils.throwIf(!updateResult, ErrorCode.OPERATION_ERROR, "更新应用部署信息失败");  
    // 10. 构建应用访问 URL    String appDeployUrl = String.format("%s/%s/", deployHost, deployKey);  
    // 11. 异步生成截图并更新应用封面  
    generateAppScreenshotAsync(appId, appDeployUrl);  
    // 部署成功 — 记录指标  
    businessMetricsCollector.recordDeployment(codeGenType, "success");  
    businessMetricsCollector.recordDeploymentDuration(codeGenType, "success",  
            java.time.Duration.ofNanos(System.nanoTime() - startNanos));  
    return appDeployUrl;  
    } catch (Exception e) {  
        // 部署失败 — 记录指标后重新抛出  
        businessMetricsCollector.recordDeployment(codeGenType, "failed");  
        businessMetricsCollector.recordDeploymentDuration(codeGenType, "failed",  
                java.time.Duration.ofNanos(System.nanoTime() - startNanos));  
        throw e;  
    }  
  
}
```

### 应用代码下载

- 校验
- 找文件目录
- 设置响应头
- `HuTool工具 压缩文件方法调用`&`输出到响应流response` 

```java
@GetMapping("/download/{appId}")  
public void downloadAppCode(@PathVariable Long appId,  
                            HttpServletRequest request,  
                            HttpServletResponse response) {  
    // 1. 基础校验  
    ThrowUtils.throwIf(appId == null || appId <= 0, ErrorCode.PARAMS_ERROR, "应用ID无效");  
    // 2. 查询应用信息  
    App app = appService.getById(appId);  
    ThrowUtils.throwIf(app == null, ErrorCode.NOT_FOUND_ERROR, "应用不存在");  
    // 3. 权限校验：只有应用创建者可以下载代码  
    User loginUser = userService.getLoginUser(request);  
    if (!app.getUserId().equals(loginUser.getId())) {  
        throw new BusinessException(ErrorCode.NO_AUTH_ERROR, "无权限下载该应用代码");  
    }  
    // 3.5. 下载前校验：只有已部署（生成了浏览地址）的应用才允许下载  
    ThrowUtils.throwIf(StrUtil.isBlank(app.getDeployKey()),  
            ErrorCode.OPERATION_ERROR, "应用尚未部署，请先生成浏览地址后再下载");  
    // 4. 构建应用代码目录路径（生成目录，非部署目录）  
    String codeGenType = app.getCodeGenType();  
    String sourceDirName = codeGenType + "_" + appId;  
    String sourceDirPath = AppConstant.CODE_OUTPUT_ROOT_DIR + File.separator + sourceDirName;  
    // 5. 检查代码目录是否存在  
    File sourceDir = new File(sourceDirPath);  
    ThrowUtils.throwIf(!sourceDir.exists() || !sourceDir.isDirectory(),  
            ErrorCode.NOT_FOUND_ERROR, "应用代码不存在，请先生成代码");  
    // 6. 生成下载文件名（不建议添加中文内容）  
    String downloadFileName = String.valueOf(appId);  
    // 7. 调用通用下载服务  
    projectDownloadService.downloadProjectAsZip(sourceDirPath, downloadFileName, response);  
}
```


```java
public void downloadProjectAsZip(String projectPath, String downloadFileName, HttpServletResponse response) {  
    // 基础校验  
    ThrowUtils.throwIf(StrUtil.isBlank(projectPath), ErrorCode.PARAMS_ERROR, "项目路径不能为空");  
    ThrowUtils.throwIf(StrUtil.isBlank(downloadFileName), ErrorCode.PARAMS_ERROR, "下载文件名不能为空");  
    File projectDir = new File(projectPath);  
    ThrowUtils.throwIf(!projectDir.exists(), ErrorCode.NOT_FOUND_ERROR, "项目目录不存在");  
    ThrowUtils.throwIf(!projectDir.isDirectory(), ErrorCode.PARAMS_ERROR, "指定路径不是目录");  
    log.info("开始打包下载项目: {} -> {}.zip", projectPath, downloadFileName);  
    // 设置 HTTP 响应头  
    response.setStatus(HttpServletResponse.SC_OK);  
    response.setContentType("application/zip");  
    response.addHeader("Content-Disposition",  
            String.format("attachment; filename=\"%s.zip\"", downloadFileName));  
    // 定义文件过滤器  
    FileFilter filter = file -> isPathAllowed(projectDir.toPath(), file.toPath());  
    try {  
        // 使用 Hutool 的 ZipUtil 直接将过滤后的目录压缩到响应输出流  
        ZipUtil.zip(response.getOutputStream(), StandardCharsets.UTF_8, false, filter, projectDir);  
        log.info("项目打包下载完成: {}", downloadFileName);  
    } catch (Exception e) {  
        log.error("项目打包下载异常", e);  
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "项目打包下载失败");  
    }  
}
```



