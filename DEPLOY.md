# 📘 GitHub 部署操作指导（英语乐园）

> 老吴的英语乐园 App 已经做完了，每次修复完只需要把最新的 `index.html` 推送到 GitHub，网站就自动更新。下面分两种情况：

---

## 🅰️ 情况一：还没创建过 GitHub 仓库（第一次部署）

按这 5 步做一次，往后所有更新都是"情况二"。

### 第 1 步：注册/登录 GitHub

打开 https://github.com/login
- 有账号 → 直接登录
- 没账号 → 点右上角 **Sign up**，填邮箱 + 密码 + 用户名（用户名会出现在最终链接里）

### 第 2 步：创建新仓库

1. 登录后点右上角 **`+`** 号 → **`New repository`**
2. 填写：

   | 字段 | 填什么 |
   |------|--------|
   | **Repository name** | `english-learn`（或 `english`、`kids-english`，但别用中文） |
   | **Description** | 留空也行 |
   | **Public / Private** | ⚠️ **`Public`**（GitHub Pages 免费版只对 Public 开放） |
   | **Add README** | ☐ 不勾选 |
   | **Add .gitignore** | None |
   | **Add license** | None |

3. 点 **`Create repository`**

### 第 3 步：上传两个文件

跳到空仓库页面，**两种上传方式任选其一**：

#### 方式 A：网页拖拽（最简单，推荐新手）

1. 在空仓库页面找 **`uploading an existing file`** 链接
2. 把这两个文件一起拖进虚线框：
   - `index.html`
   - `README.md`
3. 页面最下面 **`Commit changes`** 区域，标题填 `initial commit`
4. 点绿色 **`Commit changes`** 按钮

✅ 完成。仓库里就有这两个文件了。

#### 方式 B：Mac 终端命令（如果习惯命令行）

打开 Mac 终端（按 `Cmd + 空格` 搜"终端"），逐行执行：

```bash
# 1. 进入项目目录
cd ~/Downloads/FromCowork/Projects/英语互动学习

# 2. 初始化 git
git init

# 3. 所有文件加入暂存
git add .

# 4. 提交
git commit -m "initial commit"

# 5. 关联 GitHub 仓库（把 <你的用户名> 换成实际的用户名）
git remote add origin https://github.com/<你的用户名>/english-learn.git

# 6. 推送到 main 分支
git branch -M main
git push -u origin main
```

> ⚠️ 如果用 HTTPS 推送，第一次会要求输入 GitHub 用户名 + **Personal Access Token**（不是登录密码）。获取 token：GitHub → 头像 → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token → 勾选 `repo` → 生成 → **复制保存**，关闭窗口就再也看不到。

### 第 4 步：开启 GitHub Pages

1. 在仓库页面点 **`Settings`** 标签
2. 左侧菜单找 **`Pages`**（在 "Code and automation" 一栏）
3. **Build and deployment**：

   | 选项 | 选择 |
   |------|------|
   | Source | `Deploy from a branch` |
   | Branch | `main`，目录选 **`/ (root)`** |

4. 点 **`Save`**

### 第 5 步：等部署完成，拿链接

回到 **Settings → Pages**，等 1-2 分钟，会看到：

```
✅ Your site is live at https://<你的用户名>.github.io/english-learn/
```

把这个链接：
- 复制到老吴的手机、iPad、电脑浏览器里打开 = 完成
- 发给家人 = 孩子也能用

---

## 🅱️ 情况二：已经部署过，现在要更新（最常见）

老吴每次修完 bug 我都会更新 `index.html` 和 README.md，老吴只需要把工作目录里的新版本推到 GitHub 就好。

### 方式 A：网页拖拽（最快，30 秒搞定）

1. 打开老吴的仓库页面：https://github.com/`<你的用户名>`/english-learn
2. 找到要更新的文件（比如 `index.html`），点进去
3. 右上角 ✏️ **铅笔图标**（Edit this file）
4. 或者更快：点 **`Add file`** → **`Upload files`**，把新版本拖进去覆盖
5. 页面底部 **`Commit changes`** 标题填 `update`（或中文"修复xxx"）
6. 点 **`Commit changes`**

✅ **GitHub Pages 会自动在 1-2 分钟内重新部署**，链接不变，直接刷新就能看到新版。

### 方式 B：Mac 终端命令（推荐习惯命令行的）

每次修复完，老吴只需要执行这 3 条命令：

```bash
# 1. 进入项目目录
cd ~/Downloads/FromCowork/Projects/英语互动学习

# 2. 提交更新
git add .
git commit -m "修复xxx"

# 3. 推送到 GitHub
git push
```

然后等 1-2 分钟，刷新老吴的 Pages 链接即可。

> 💡 **如果第一次推送时没用过 git 推送**，现在想用：需要先执行一次情况一的 Step 3 方式 B 第 2-6 步（git init / add / commit / remote add / push）。之后再更新就用上面的 3 条命令即可。

---

## 🔄 更新后验证（强烈建议）

不管用哪种方式更新，**都建议在浏览器里验证一下**：

1. 打开老吴的 Pages 链接（带隐身模式打开能跳过 localStorage 缓存）
2. 看 5 个模块 tab 都能切
3. 玩 1-2 个游戏确认修复点生效
4. 在安卓手机上试一下发音（如果有）

---

## ❓ 常见问题速查

| 问题 | 解决 |
|------|------|
| 链接打开是 404 | 等 2-3 分钟再刷；Pages 部署需要时间 |
| 终端推送要求输入密码 | 用 Personal Access Token，不要用 GitHub 登录密码 |
| 想换电脑/重装系统后再推送 | clone 一下仓库：`git clone https://github.com/<用户名>/english-learn.git`，以后照常 push 即可 |
| 推送时报 "Updates were rejected" | 老吴的本地版本比 GitHub 旧，先 `git pull origin main` 再 `git push` |
| 多人合作时合并冲突 | 这种情况老吴一个人用应该不会遇到，遇到了告诉我 |
| 想绑定自己的域名（短一些好记） | 在仓库根目录新建 `CNAME` 文件（无后缀），内容写你的域名，然后去域名服务商加 CNAME 记录指向 `<用户名>.github.io.` |
| 想清空老吴本地进度 | 浏览器开发者工具 → Application → Storage → Clear site data |
| 怕每次推错搞坏线上版本 | 在仓库 Settings → General → 拉到最下 Danger Zone → Archive this repository（归档后只读但保留）|

---

## 🎯 老吴的 30 秒速查卡

```
日常更新三板斧（终端）：
  cd ~/Downloads/FromCowork/Projects/英语互动学习
  git add . && git commit -m "update" && git push

日常更新网页版：
  仓库 → Add file → Upload files → 拖新文件 → Commit

更新完等 2 分钟，刷新 Pages 链接。
```

---

*文档版本：1.0 · 2026-07-26*