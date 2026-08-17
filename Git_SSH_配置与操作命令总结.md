# Git + SSH 配置与操作命令总结

## 一、Git + SSH 建立方法（以 GitHub 为例，GitLab/Gitee 同理）

### 1. 检查是否已有 SSH 密钥
```bash
ls -al ~/.ssh
```
如果看到 `id_ed25519` / `id_ed25519.pub`（或 `id_rsa`），说明已生成过，可跳过第 2 步。

### 2. 生成 SSH 密钥
```bash
ssh-keygen -t ed25519 -C "你的邮箱@example.com"
```
一路回车即可（也可为密钥设置密码口令）。会生成一对密钥：
- 私钥 `~/.ssh/id_ed25519`（**绝不能泄露**）
- 公钥 `~/.ssh/id_ed25519.pub`（发给托管平台）

### 3. 启动 ssh-agent 并添加私钥
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### 4. 把公钥添加到托管平台
```bash
cat ~/.ssh/id_ed25519.pub
```
复制输出内容，然后：
- **GitHub**：Settings → SSH and GPG keys → New SSH key → 粘贴
- **Gitee**：设置 → SSH公钥 → 粘贴
- **GitLab**：Preferences → SSH Keys → 粘贴

### 5. 测试连接
```bash
ssh -T git@github.com     # GitHub
ssh -T git@gitee.com      # Gitee
ssh -T git@gitlab.com     # GitLab
```
看到 `Hi xxx! You've successfully authenticated...` 即成功。

### 6.（可选）多账号配置 `~/.ssh/config`
```bash
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github

Host gitee.com
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitee
```

### 7. 把已有仓库的远程地址从 HTTPS 改为 SSH
```bash
git remote set-url origin git@github.com:用户名/仓库名.git
git remote -v   # 验证
```

> **Windows 提示**：在 Git Bash 中操作最方便；配置文件位置为 `C:\Users\你的用户名\.ssh\`。若 `ssh-keygen` 不是内部命令，说明未安装 Git for Windows。

---

## 二、Git 常用命令说明

### 1. 配置（首次使用）
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global --list          # 查看配置
```

### 2. 初始化 / 克隆
```bash
git init                             # 在当前目录创建仓库
git clone git@github.com:user/repo.git   # 克隆远程仓库（SSH 方式）
```

### 3. 核心工作流（最常用）
| 命令 | 作用 |
|------|------|
| `git status` | 查看工作区状态 |
| `git add 文件` / `git add .` | 将改动加入暂存区 |
| `git commit -m "说明"` | 提交暂存区内容到本地仓库 |
| `git push` | 推送到远程 |
| `git pull` | 拉取远程更新并合并 |

完整流程：**改代码 → `add` → `commit` → `push`**

### 4. 分支管理
```bash
git branch                     # 查看本地分支
git branch 分支名               # 新建分支
git checkout 分支名             # 切换分支（旧写法）
git switch 分支名               # 切换分支（新写法，更直观）
git switch -c 分支名            # 新建并切换
git branch -d 分支名            # 删除分支
git branch -a                  # 查看所有分支（含远程）
```

### 5. 合并与解决冲突
```bash
git merge 分支名                # 把某分支合并进当前分支
git rebase 分支名               # 变基（历史更线性，慎用）
```
合并冲突时，Git 会标出冲突文件，手动修改后：
```bash
git add 冲突文件
git commit -m "解决冲突"
```

### 6. 撤销 / 回退
```bash
git restore 文件                # 丢弃工作区改动（未 add）
git restore --staged 文件       # 取消暂存（保留改动）
git reset --hard HEAD~1         # 回退到上一个提交并丢弃改动（危险）
git reset --soft HEAD~1         # 回退提交但保留改动
git log --oneline               # 查看提交历史，找版本号
```

### 7. 远程仓库
```bash
git remote -v                   # 查看远程地址
git remote add origin 地址       # 添加远程
git remote set-url origin 新地址 # 修改远程地址
```

### 8. 查看历史与差异
```bash
git log --oneline --graph --all # 图形化查看历史
git diff                        # 查看未暂存改动
git diff --staged               # 查看已暂存改动
git show 版本号                  # 查看某次提交详情
```

### 9. 标签（打版本）
```bash
git tag v1.0                    # 打标签
git push origin v1.0            # 推送标签
```

---

## 三、速查口诀
- **改完三连**：`add` → `commit` → `push`
- **换代码先**：`git pull`（避免冲突）
- **出错不怕**：`git status` 永远是第一步
