---
name: i18n-translator
description: 将前端代码改造成翻译脚本可识别的格式。触发词：i18n、国际化、中文翻译。
---

# 前端中文改造规则

## 核心原则

**只改造翻译脚本识别不到的中文**，不改变业务逻辑。

翻译脚本能识别：**字符串字面量**（用引号包裹的中文）
翻译脚本识别不到：**JSX 文本节点**（无引号包裹）、**模板字符串中的复杂表达式**

---

## 一、不需要改造的情况

### 1. 注释中的中文

```tsx
// 单行注释
/* 多行注释 */
{/* JSX 注释 */}
```

### 2. 字符串字面量

**以下情况中被单引号或双引号包裹的中文**，翻译脚本能识别，不允许改：

```tsx
//  JSX 属性值可以被脚本识别 - 不允许改
<Spin tip="加载中..." />
<Input placeholder="扫描/选择 工单" />
<Button icon={<Icon />}>{'搜索'}</Button>

//  函数参数 - 不允许改
message.error('操作失败');
message.success('保存成功');
throw new Error('工单不能为空');

// 对象属性值 - 不允许改
const columns = [{ title: '生产任务编号', dataIndex: 'code' }];
const config = { label: '状态', value: 1 };

// 变量赋值 - 不允许改
const text = '确认删除';
```

### 3. JSX 表达式中的字符串字面量

**已被 `{}` 包裹的表达式**，其中的字符串字面量不需要改：

```tsx
// 已经是 JSX 表达式 - 不需要改
{expand ? '收起' : '展开'}
{isActive ? '进行中' : '已完成'}
{someFlag && '显示文本'}

// 花括号中的字符串 - 不需要改
<span>{'已有引号，不需要改'}</span>
<div>{`已有模板字符串，不需要改`}</div>
```

### 4. 模板字符串中的简单变量

模板字符串中**只有简单变量引用**，不需要改：

```tsx
// ✅ 简单变量 - 不需要改
const status = '启用';
message.success(`已${status}`);

const name = '张三';
const text = `${name}的工单`;
```

---

## 二、需要改造的情况

### 情况 1：JSX 文本节点（无 `{}` 包裹）

**判断方法**：标签之间的文本，没有用 `{}` 包裹

```tsx
// ❌ 需要改造
<span>生产</span>
<div>这是文本节点</div>
<a>查看详情</a>

// ✅ 改造后
<span>{'生产'}</span>
<div>{'这是文本节点'}</div>
<a>{'查看详情'}</a>
```

#### 带嵌套元素的文本节点

```tsx
// ❌ 需要改造
<span>删除后<span style={{ color }}>不可恢复</span>，是否确认删除？</span>

// ✅ 改造后
<span>
  {'删除后'}
  <span style={{ color }}>{'不可恢复'}</span>
  {'，是否确认删除？'}
</span>
```

#### 文本节点与变量混合

```tsx
// ❌ 需要改造
<div>
  节点ID：{config.code}
  执行频率：{getContent(config.timeJob.execType)}
</div>

// ✅ 改造后
<div>
  {'节点ID：'}
  {config.code}
  {(() => {
    const T1 = getContent(config.timeJob.execType);
    return <>{'执行频率：'}{T1}</>;
  })()}
</div>
```

---

### 情况 2：模板字符串中的复杂表达式

模板字符串的 `${}` 中包含**函数调用、对象访问、三元表达式**等，需要改造：

#### 函数调用

```tsx
// ❌ 需要改造
const text = `${getCount()}条`;

// ✅ 改造后
const T1 = getCount();
const text = `${T1}条`;
```

#### 对象/属性访问

```tsx
// ❌ 需要改造
const text = `${appEnum.common.yn.yes}个`; // appEnum.common.yn.yes = '是'
const msg = `${user.name}的操作记录`;

// ✅ 改造后
const T1 = appEnum.common.yn.yes;
const text = `${T1}个`;

const T2 = user.name;
const msg = `${T2}的操作记录`;
```

#### 三元表达式

```tsx
// ❌ 需要改造
title={`${lookup('common.crud')(mode)}分析告警规则`}

// ✅ 改造后
const T1 = lookup('common.crud')(mode);
title={`${T1}分析告警规则`});
```

#### 复杂嵌套

```tsx
// ❌ 需要改造
name: mode === CRUD.copy ? `${initialInfo.name}_复制` : initialInfo.name,

// ✅ 改造后
name: (() => {
  const T1 = initialInfo.name;
  return mode === CRUD.copy ? `${T1}_复制` : T1;
})(),
```

---

## 快速判断表

| 代码形式 | 是否需要改 | 判断依据 |
|---------|-----------|---------|
| `<span>文本</span>` | ✅ 需要 | JSX 文本节点，无引号 |
| `<span>{'文本'}</span>` | ❌ 不需要 | 已是 JSX 表达式 |
| `<Input placeholder="文本" />` | ❌ 不需要 | JSX 属性值（字符串字面量） |
| `message.error('文本')` | ❌ 不需要 | 函数参数（字符串字面量） |
| `throw new Error('文本')` | ❌ 不需要 | 函数参数（字符串字面量） |
| `{flag ? 'A' : 'B'}` | ❌ 不需要 | 已是 JSX 表达式 |
| `` `${var}文本` `` | ❌ 不需要 | 模板字符串中只有简单变量 |
| `` `${func()}文本` `` | ✅ 需要 | 模板字符串中有函数调用 |
| `` `${obj.prop}文本` `` | ✅ 需要 | 模板字符串中有对象访问 |

---

## 改造示例

### 示例 1：组件中的文本节点

```tsx
// 改造前
const Header = () => {
  return (
    <div className="header">
      <h1>生产任务列表</h1>
      <button>新建任务</button>
      {showDetail && <span>任务详情</span>}
    </div>
  );
};

// 改造后
const Header = () => {
  return (
    <div className="header">
      <h1>{'生产任务列表'}</h1>
      <button>{'新建任务'}</button>
      {showDetail && <span>{'任务详情'}</span>}
    </div>
  );
};
```

### 示例 2：模板字符串

```tsx
// 改造前
const getMessage = () => {
  return `${user.name}在${formatTime(date)}完成了${task.name}任务`;
};

// 改造后
const getMessage = () => {
  const T1 = user.name;
  const T2 = formatTime(date);
  const T3 = task.name;
  return `${T1}在${T2}完成了${T3}任务`;
};
```

### 示例 3：混合情况

```tsx
// 改造前
const Card = ({ data }) => {
  return (
    <div>
      <h3>{data.title}</h3>
      <p>
        状态：{data.status}
        进度：${calculateProgress(data)}%
      </p>
      <button onClick={() => handleClick(data.id)}>
        {data.canEdit ? '编辑' : '查看'}
      </button>
    </div>
  );
};

// 改造后
const Card = ({ data }) => {
  return (
    <div>
      <h3>{data.title}</h3>
      <p>
        {'状态：'}{data.status}
        {(() => {
          const T1 = calculateProgress(data);
          return <>{'进度：'}{T1}%</>;
        })()}
      </p>
      <button onClick={() => handleClick(data.id)}>
        {data.canEdit ? '编辑' : '查看'}
      </button>
    </div>
  );
};
```

---

## 注意事项

1. **不要过度改造**：字符串字面量不需要改，避免增加不必要的花括号
3. **不要改变业务逻辑**：只是改变代码结构，不改变功能
4. **注释不改**：保留所有中文注释
