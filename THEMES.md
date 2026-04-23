# 🎨 Hugo 主题管理指南

## 当前使用主题

**PaperMod** — 简洁、快速、功能丰富

---

## 🌟 推荐主题列表

### 博客向（适合日常写作）

| 主题名 | 风格 | 特色 | GitHub |
|--------|------|------|--------|
| **PaperMod** | 简洁极简 | 内置搜索、目录、代码复制、暗色模式 | `adityatelange/hugo-PaperMod` |
| **Stack** | 卡片式 | 左侧导航、双栏布局、归档日历 | `CaiJimmy/hugo-theme-stack` |
| **Congo** | 现代清新 | 多布局切换、Tailwind CSS | `jpanther/congo` |
| **Blowfish** | 现代设计 | Firebase 实时访客、多语言 | `nunocoracao/blowfish` |
| **Ananke** | 经典博客 | Hugo 官方推荐入门主题 | `theNewDynamic/gohugo-theme-ananke` |

### 技术文档向（适合教程/代码）

| 主题名 | 风格 | 特色 | GitHub |
|--------|------|------|--------|
| **Docsy** | Google 文档风 | 版本管理、多级导航 | `google/docsy` |
| **Book** | 书籍排版 | 目录树、适合长篇教程 | `alex-shpak/hugo-book` |
| **Geekdoc** | 技术极客风 | 代码块优化、搜索 | `thegeeklab/hugo-geekdoc` |

---

## 🔄 如何切换主题（3步完成）

### 方法一：Git Submodule（推荐，持续接收主题更新）

```bash
# 第一步：删除旧主题
git submodule deinit themes/OLD_THEME
git rm themes/OLD_THEME

# 第二步：添加新主题
git submodule add https://github.com/CaiJimmy/hugo-theme-stack themes/Stack

# 第三步：修改 hugo.toml
# 将 theme = "PaperMod" 改为 theme = "Stack"
```

### 方法二：直接克隆（适合快速体验）

```bash
git clone https://github.com/CaiJimmy/hugo-theme-stack themes/Stack
# 修改 hugo.toml：theme = "Stack"
```

### 方法三：Hugo Modules（高级，模块化管理）

```bash
hugo mod init github.com/LinFei26/linfei-blog

# hugo.toml 中添加
[module]
  [[module.imports]]
    path = "github.com/CaiJimmy/hugo-theme-stack/v3"
```

---

## ⚠️ 切换主题注意事项

1. **Front Matter 参数** — 不同主题对文章头部参数支持不同，切换后检查 `hugo.toml` 的 `[params]` 部分
2. **菜单配置** — `[[menu.main]]` 在多数主题通用，但部分主题用自己的菜单格式
3. **封面图片** — `cover.image` 字段名称在不同主题可能不同
4. **本地预览** — 切换后运行 `hugo server` 本地验证再提交

---

## 🎨 如何自定义主题（不修改主题源码）

Hugo 支持**主题覆盖（Theme Override）**：

```
linfei-blog/
├── layouts/          ← 覆盖主题的模板
│   ├── partials/
│   └── _default/
├── static/           ← 覆盖/追加静态资源
└── assets/           ← 覆盖 CSS/JS
```

---

## 🧪 本地预览命令

```bash
hugo server -D --port 1313
# 访问 http://localhost:1313
```
