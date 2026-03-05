# translator skill 使用说明

## 什么是 translator？

translator 是一个 Claude Code skill，用于执行完整的多语言翻译流程。

## 如何调用

当需要翻译时，直接说：
- "/translator"
- "翻译"
- "多语言翻译"
- "i18n"

## 完整流程

```
1. 改造代码 → 翻译脚本可识别的格式
2. 检查是否首次接入
3. 初始化配置（仅首次）
4. 提取中文并替换
5. 上传翻译内容
6. 下载多语言文件
7. 检查并提交
```

## 翻译范围指定

### 方式一：目录
```
翻译 src/page/productionPlanning/machineProcessing
```

### 方式二：文件
```
翻译 src/page/productionPlanning/machineProcessing/pages/home/index.tsx
```

### 方式三：从某个 commit 到最新
```bash
翻译 <folder>  <commit>
```

注意：当从某个 commit 到最新的翻译没有指定<folder>,会默认翻译范围是src路径下
