# LinFei's Blog — Hugo 站点

> 部署于 [linfei26.github.io](https://linfei26.github.io)  
> 机器人技术 | 游戏设计 | 日常随笔

---

## 🚀 快速开始

### 第一次使用（本地环境搭建）

**1. 安装 Hugo Extended**

```bash
# Windows（推荐 Scoop）
scoop install hugo-extended

# macOS
brew install hugo

# 验证安装
hugo version
```

**2. 克隆仓库并安装主题**

```bash
git clone --recurse-submodules https://github.com/LinFei26/linfei26.github.io.git
cd linfei26.github.io

# 如果没有 --recurse-submodules，手动初始化主题
git submodule update --init --recursive
```

**3. 安装默认主题 PaperMod**

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

**4. 本地预览**

```bash
hugo server -D
# 访问 http://localhost:1313
```

---

## ✍️ 内容创作

### 三大内容板块

| 板块 | 命令 | 用途 |
|------|------|------|
| 🤖 机器人 | `hugo new content robot/xxx.md` | ROS2、算法、开源项目 |
| 🎮 游戏 | `hugo new content game/xxx.md` | 游戏设计学习记录 |
| 💬 博客 | `hugo new content blog/xxx.md` | 随笔、闲聊 |

> 命令会自动使用 `archetypes/` 中对应的模板填充 Front Matter。

---

## 📁 项目结构

```
linfei-blog/
├── .github/
│   └── workflows/
│       └── deploy.yml        # 自动部署 CI/CD
├── archetypes/               # 内容模板
│   ├── robot.md
│   ├── game.md
│   └── blog.md
├── content/                  # 所有文章
│   ├── robot/                # 机器人板块（含 tutorials/、snippets/）
│   ├── game/                 # 游戏板块
│   ├── blog/                 # 博客板块
│   └── about.md              # 关于页
├── static/
│   └── images/               # 图片资源
├── themes/                   # Hugo 主题（git submodule）
│   └── PaperMod/
├── hugo.toml                 # 站点配置
├── THEMES.md                 # 主题切换指南
└── README.md
```

---

## 🌐 部署到 GitHub Pages

### 首次部署（一次性操作）

1. **在 GitHub 创建仓库**：名称必须为 `linfei26.github.io`

2. **推送代码**
   ```bash
   git init
   git add .
   git commit -m "init: Hugo blog"
   git branch -M main
   git remote add origin https://github.com/LinFei26/linfei26.github.io.git
   git push -u origin main
   ```

3. **开启 GitHub Pages**  
   仓库 → Settings → Pages → Source 选择 **GitHub Actions**

### 后续发布

```bash
git add .
git commit -m "post: 文章标题"
git push
# GitHub Actions 自动构建部署，2分钟内上线
```

---

## 🎨 切换主题

详见 [THEMES.md](./THEMES.md) — 包含 8 款推荐主题和完整切换步骤。
