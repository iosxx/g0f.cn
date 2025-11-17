# 故障排查指南

## 问题：提交 Issue 后没有自动打标签

### 原因分析
1. **Issue 模板未正确识别**：GitHub 需要一段时间来识别新的或更新的 Issue 模板
2. **未使用模板创建 Issue**：如果直接点击 "New Issue" 而不是选择模板，标签不会自动添加
3. **仓库设置问题**：确保 Issue 功能已启用

### 解决方案
1. **使用正确的方式创建 Issue**：
   - 访问：https://github.com/Jiosanity/xiaoten-links/issues/new/choose
   - 选择 "友链申请" 模板
   - 填写表单后提交

2. **手动添加标签**（临时方案）：
   - 在 Issue 页面右侧找到 "Labels"
   - 添加 `friendlink` 和 `pending` 标签

3. **验证模板文件**：
   - 确保 `.github/ISSUE_TEMPLATE/friendlink.yml` 文件已推送到 GitHub
   - 文件名必须是 `.yml` 扩展名

## 问题：评论 `/approve` 没有反应

### 原因分析
1. **GitHub Actions 未启用**
2. **工作流权限不足**
3. **Issue 缺少 `friendlink` 标签**
4. **工作流文件有语法错误**

### 解决方案

#### 步骤 1：检查 GitHub Actions 是否启用
1. 访问：https://github.com/Jiosanity/xiaoten-links/actions
2. 如果显示 "Actions are disabled"，点击 "Enable Actions"
3. 确保允许所有 Actions 运行

#### 步骤 2：检查工作流权限
1. 访问：https://github.com/Jiosanity/xiaoten-links/settings/actions
2. 在 "Workflow permissions" 部分：
   - 选择 "Read and write permissions"
   - 勾选 "Allow GitHub Actions to create and approve pull requests"
3. 点击 "Save"

#### 步骤 3：验证 Issue 标签
确保你的 Issue 包含以下标签：
- ✅ `friendlink` 标签（必需）
- 🔄 `pending` 标签（可选）

如果没有，手动添加 `friendlink` 标签。

#### 步骤 4：测试工作流
1. 确保 Issue 有 `friendlink` 标签
2. 评论 `/approve`（注意是小写）
3. 等待几秒钟，`approve-application` job 会添加 `approved` 标签
4. **添加标签后会自动触发 `process-approved-application` job**
5. 查看 Actions 运行情况：https://github.com/Jiosanity/xiaoten-links/actions
6. 应该会看到两个工作流运行：
   - 第一个：评论触发，运行 `approve-application`
   - 第二个：标签触发，运行 `process-approved-application`

#### 步骤 5：查看工作流日志
如果工作流运行了但失败：
1. 访问 Actions 页面
2. 点击失败的工作流
3. 查看详细日志找出错误原因

### 常见错误

#### 错误 1：权限不足
```
Error: Resource not accessible by integration
```
**解决方案**：按照步骤 2 设置正确的权限

#### 错误 2：找不到标签
**症状**：评论后没有任何反应
**解决方案**：
1. 检查 Issue 是否有 `friendlink` 标签
2. 标签名称必须完全匹配（区分大小写）

#### 错误 3：工作流未触发
**症状**：Actions 页面没有新的运行记录
**可能原因**：
1. GitHub Actions 未启用
2. 工作流文件有语法错误
3. Issue 没有 `friendlink` 标签

#### 错误 4：第二个 job 被跳过
**症状**：`approve-application` 运行成功，但 `process-approved-application` 显示 "skipped"
**原因**：第二个 job 只在添加 `approved` 标签时触发，需要等待标签事件
**解决方案**：
1. 评论 `/approve` 后，等待第一个工作流完成
2. 第一个工作流会添加 `approved` 标签
3. 添加标签会触发第二个工作流运行
4. 在 Actions 页面应该能看到两个独立的工作流运行记录

## 完整的工作流程

### 正确的操作流程

1. **提交 Issue**：
   ```
   访问：https://github.com/Jiosanity/xiaoten-links/issues/new/choose
   选择 "友链申请" 模板
   填写所有必填字段
   提交 Issue
   ```

2. **验证自动标签**：
   - Issue 应该自动带有 `friendlink` 和 `pending` 标签
   - 如果没有，手动添加

3. **审批流程**（仓库维护者）：
   ```
   在 Issue 中评论：/approve
   ```

4. **自动处理**：
   - GitHub Actions 自动添加 `approved` 标签
   - 移除 `pending` 标签
   - 解析 Issue 内容
   - 更新 `links.json`
   - 创建 Pull Request
   - 添加确认评论

5. **合并 PR**：
   - 检查 PR 中的更改
   - 合并 PR 到主分支

## 快速诊断命令

### 检查工作流文件语法
在本地运行：
```bash
# 安装 actionlint（可选）
# https://github.com/rhysd/actionlint

actionlint .github/workflows/process-friendlink.yml
```

### 手动触发工作流测试
如果需要手动测试，可以暂时修改工作流添加 `workflow_dispatch`：

```yaml
on:
  workflow_dispatch:  # 添加这一行
  issues:
    types: [opened, edited, labeled]
  issue_comment:
    types: [created]
```

## 需要帮助？

如果以上方法都不能解决问题，请：

1. **检查 Actions 日志**：
   - 访问：https://github.com/Jiosanity/xiaoten-links/actions
   - 查看失败的工作流日志

2. **提供详细信息**：
   - Issue 链接
   - 错误日志截图
   - Actions 运行日志

3. **验证清单**：
   - [ ] GitHub Actions 已启用
   - [ ] 工作流权限设置为 "Read and write"
   - [ ] Issue 有 `friendlink` 标签
   - [ ] 使用模板创建的 Issue
   - [ ] 评论是 `/approve`（小写）
