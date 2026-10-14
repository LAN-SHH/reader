# 如何提交 PR 到 Reader 项目

## 已完成的修改

我已经在本地完成了以下修复：

### 1. 问题分析
- **根本原因**: BetterBox 等代理软件设置的 bypass 规则包含 `172.2*` 和 `*jd.com` 等格式
- **现有问题**: Reader 的解析器不支持这些格式，导致系统代理读取失败
- **影响**: 用户无法使用系统代理，必须手动配置

### 2. 修复内容
- ✅ 支持部分 IP 通配符：`172.2*` → `172.20.*.*` 到 `172.29.*.*`
- ✅ 支持域名通配符：`*jd.com` → `jd.com`（NO_PROXY 格式为 `.jd.com`）
- ✅ 改进错误日志：显示具体错误信息而不是吞掉异常
- ✅ 添加测试用例

### 3. 修改的文件
- `apps/desktop/main/windows-proxy.ts` - 主要修复逻辑
- `apps/desktop/main/proxy.ts` - 改进错误日志
- `apps/desktop/main/windows-proxy.test.ts` - 添加测试用例
- `PR_DESCRIPTION.md` - PR 说明文档

## 提交 PR 步骤

### 步骤 1: Fork 仓库

1. 访问 https://github.com/cynicalight/reader
2. 点击右上角的 **Fork** 按钮
3. Fork 到你自己的 GitHub 账号

### 步骤 2: 添加你的 fork 作为 remote

```bash
cd ~/reader
git remote add fork https://github.com/YOUR_USERNAME/reader.git
```

（把 `YOUR_USERNAME` 替换成你的 GitHub 用户名）

### 步骤 3: 推送分支到你的 fork

```bash
git push fork fix/windows-proxy-bypass-parsing
```

### 步骤 4: 创建 Pull Request

1. 访问你 fork 的仓库：`https://github.com/YOUR_USERNAME/reader`
2. 会看到提示："fix/windows-proxy-bypass-parsing had recent pushes"
3. 点击 **Compare & pull request** 按钮
4. 填写 PR 信息：

**标题:**
```
fix: support partial wildcard and domain wildcard in Windows proxy bypass
```

**描述:** (复制 `PR_DESCRIPTION.md` 的内容)

### 步骤 5: 等待审核

项目维护者会审核你的 PR，可能会：
- 直接合并
- 要求修改
- 提出问题

## 快速命令汇总

```bash
# 1. Fork 后添加 remote（只需执行一次）
cd ~/reader
git remote add fork https://github.com/YOUR_USERNAME/reader.git

# 2. 推送分支
git push fork fix/windows-proxy-bypass-parsing

# 3. 在 GitHub 网页上创建 PR
```

## 临时解决方案（给你自己用）

在 PR 被合并之前，你有两个选择：

### 选项 1: 使用修改后的系统代理（我已经帮你设置了）
```bash
# 我已经把你的 bypass 列表中的 172.2* 展开了
# 直接重启 Reader 应该就能读取系统代理
```

### 选项 2: 手动配置代理
- 设置 → 诊断 → 代理
- 选择 "手动 HTTP 代理"
- 地址: 127.0.0.1，端口: 7890
- 保存并重启

## 验证修复

重启 Reader 后：
1. 打开 设置 → 诊断
2. 应该显示代理模式为 "手动 HTTP 代理" 或系统代理
3. 设置 → Agent → 填入 API 配置
4. 应该能正常获取模型列表

---

**注意**: 即使连接检测显示超时（因为 chatgpt.com 被墙），你的第三方 API 也能正常工作。这是正常的！
