---
name: translate
description: 执行完整的多语言翻译流程。包括：将代码改造成翻译脚本可识别的格式、检查项目首次接入、初始化配置、提取并替换中文、上传翻译内容、下载多语言文件。触发词：翻译、多语言、i18n
---

# 多语言翻译流程

## 完整流程

```
1. 将代码改造成翻译脚本可识别的格式
       ↓
2. 检查项目是否首次接入翻译脚本
       ↓
3. 如果是首次，初始化配置
       ↓
4. 提取中文并替换为脚本可识别的格式
       ↓
5. 上传新的中文信息
       ↓
6. 下载多语言 JSON 文件
       ↓
7. 提醒用户检查
```

---

## 步骤 1：将代码改造成翻译脚本可识别的格式

**首先调用 i18n-translator skill**，将目标代码中翻译脚本无法识别的中文改为可识别的格式：

- JSX 文本节点（无 `{}` 包裹）→ 改为 `{'中文'}`
- 模板字符串中的复杂表达式（函数调用、对象访问）→ 提取到变量

**参考**：i18n-translator-v3 skill 的详细规则

**重要**：必须先完成代码改造，确保所有翻译脚本无法识别的中文都已改为可识别的格式再进入下一步

---

## 步骤 2：检查项目是否首次接入

询问用户项目是否**第一次接入**翻译脚本。

判断方法：
- 项目中是否已有 i18n 文件或者相关配置（如 `intl` 国际化包、多语言配置文件等） 
- 或者直接询问用户

---

## 步骤 3：初始化配置（仅首次接入）

如果项目**首次接入**，需要执行初始化：

```bash
npx @blacklake-web/codemod init <project>
```

- `<project>` 是项目名称，通常是项目目录名

---

## 步骤 4：提取中文并替换

翻译前检查：**是否已经首先调用 i18n-translator skill**，将目标代码中所有的翻译脚本无法识别的中文改为可识别的格式了，如果还有，重新执行步骤一

询问用户想要翻译的范围：

- **整个目录**：提供目录路径
- **单个文件**：提供文件路径
- **指定提交到最新提交的所有变更内容**：提供 commit hash和文件范围

### 4.1 路径转换

**重要**：用户提供的路径可能是完整的绝对路径，但命令需要在**项目根目录**中执行。

**示例**：

- 用户提供的完整路径：`/Users/fz/Documents/轻制造/bf-execution-h5/src/page/productionPlanning/machineProcessing/pages/execution/InfoDisplay.tsx`
- 项目根目录：`/Users/fz/Documents/轻制造/bf-execution-h5/`
- transform 命令需要的路径：`src/page/productionPlanning/machineProcessing/pages/execution/InfoDisplay.tsx`

**转换方法**：将完整路径中去掉项目根目录前缀，得到相对于项目根目录的路径。

```bash
# 翻译指定目录
npx @blacklake-web/codemod transform <folder>

# 翻译指定文件
npx @blacklake-web/codemod transform <file>

# 翻译特定提交到最新提交的所有变更的内容
npx @blacklake-web/codemod transform <folder> <commit>
如果用户没有提供翻译的范围<folder>，默认翻译src路径下文件，即直接执行npx @blacklake-web/codemod transform src <commit>
```

---

## 步骤 5：上传新的中文信息

```bash
npx @blacklake-web/codemod upload
```

此命令会上传此次新增的翻译内容。

---

## 步骤 6：下载多语言 JSON

```bash
npx @blacklake-web/codemod download
```

下载翻译好的多语言 JSON 文件。

---

## 步骤 7：提醒用户检查

完成后，提醒用户：

1. **检查转换结果**：确认代码改动符合预期，没有不必要的修改
2. **检查翻译内容**：确认多语言 JSON 文件内容正确
3. **提交代码**：将改动提交到仓库

---

## 步骤 8：处理异常情况

如果 transform 后出现异常，按照以下步骤处理：

### 8.1 理解异常信息

翻译脚本会输出类似以下信息：

```
共 1 个文件, 35 处中文替换完成,4 处异常
注意！你还需要做如下工作:
1. 撤回此次所有翻译范围内的文件
2. 修复以下文件内存在的错误
src/page/productionPlanning/machineProcessing/pages/execution/InfoDisplay.tsx 异常文本：  计划生产: {undefined}{undefined}
3. 重新运行transform翻译脚本
4. 运行upload上传脚本保存到远端
5. 运行download下载脚本,最后提交此次翻译更新内容
```

### 8.2 常见异常类型

| 异常文本 | 原因 | 修复方法 |
|---------|------|---------|
| `{undefined}{undefined}` | 模板字符串中提取的变量是 undefined | 检查原始代码中的变量是否正确定义 |
| `{undefined}成功` | 模板字符串中的变量是 undefined | 检查变量来源 |
| `{undefined}失败` | 同上 | 同上 |

### 8.3 修复步骤

```bash
# 1. 撤回此次所有翻译范围内的文件
git checkout -- <files>

# 2. 修复代码中导致异常的问题
#    - 检查模板字符串中的变量是否正确提取
#    - 确保变量在作用域内已定义

# 3. 重新运行 transform
npx @blacklake-web/codemod transform <folder>

# 4. 运行 upload
npx @blacklake-web/codemod upload

# 5. 运行 download
npx @blacklake-web/codemod download
```

### 8.4 常见需要修复的代码模式

这些模式容易导致异常，**在调用 i18n-translator skill 时应该手动修复**：

```tsx
// ❌ 异常：模板字符串中的变量是 undefined
const text = `${someFunction()}结果`;
// → 先提取函数调用：const T1 = someFunction(); const text = `${T1}结果`;

// ❌ 异常：对象属性访问
const text = `${obj.key}信息`;
// → 先提取属性：const T1 = obj.key; const text = `${T1}信息`;

// ✅ 正确：简单变量可以直接用
const status = '进行中';
message.success(`${status}成功`);
```

### 8.5 重复变量问题

**当字符串模板中拼接了多个相同变量时**，transform 后会生成重复参数，导致运行时错误：

```tsx
// ❌ 原始代码
message.success(`${userName}已上线，欢迎${userName}`);

// ❌ transform 后会生成（错误）
intl.get('xxx.welcome', { userName, userName })
```

**判断方法**：检查代码中是否存在 `${foo}...${foo}` 这种模式（同一个变量出现多次）

**修复方法**：

```tsx
// ✅ 改造后：删除重复变量
message.success(`${userName}已上线，欢迎${userName}`);
// transform 后生成：
intl.get('xxx.welcome', { userName })
```

**在 transform 后检查并手动删除重复变量**，确保最终代码中不存在 `{ foo, foo }` 这种重复参数。

---

## 常见问题

### Q: 初始化命令执行失败？
A: 检查项目是否在白名单中，或者联系基础设施团队。

### Q: transform 后中文没有被替换？
A: 检查代码是否符合翻译脚本的识别规则，参考 i18n-translator skill。

### Q: upload 失败？
A: 确认是否有新增的中文内容，或者网络问题。

### Q: download 后找不到对应语言？
A: 确认目标语言是否已配置，或者等待翻译完成。

### Q: 出现 {undefined} 异常怎么办？
A: 按照步骤 8 的修复步骤处理，主要是检查模板字符串中的变量是否正确提取。

---

## 执行示例

```bash
# 假设用户想要翻译 src/page/productionPlanning/machineProcessing 目录

# 1. 先改造代码（调用 i18n-translator skill）
#    将该目录下的中文改造成翻译脚本可识别的格式

# 2. 首次接入时初始化
> npx @blacklake-web/codemod init my-project

# 3. 提取并替换
> npx @blacklake-web/codemod transform src/page/productionPlanning/machineProcessing

# 4. 上传
> npx @blacklake-web/codemod upload

# 5. 下载
> npx @blacklake-web/codemod download

# 6. 提醒用户检查并提交
```
