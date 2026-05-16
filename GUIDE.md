# Git Demo 项目 - 学习 Git 内部机制

## 项目结构

```
git-demo/
├── .git/                    # Git 仓库（核心！）
│   ├── HEAD                 # HEAD 指针文件
│   ├── refs/
│   │   └── heads/
│   │       ├── master       # master 分支指针
│   │       └── feature      # feature 分支指针
│   └── ...
├── .githooks/               # 自定义 Git 钩子（可被 Git 跟踪）
│   ├── pre-commit           # 提交前钩子
│   ├── commit-msg           # 提交信息钩子
│   ├── pre-push             # 推送前钩子
│   └── post-checkout        # 切换分支后钩子
├── index.js                 # 示例代码
└── GUIDE.md                 # 本文件
```

## 当前提交历史

```
master:   C1 -- C2
               \
feature:        C3
```

- C1: Initial commit（README + index.js）
- C2: Add subtract function（master 分支）
- C3: Add multiply function（feature 分支）

---

## 实验一：理解 HEAD 指针

### 1.1 查看 HEAD 文件内容

```bash
cat .git/HEAD
# 预期输出：ref: refs/heads/master
# 含义：HEAD 间接指向 master 分支
```

### 1.2 查看分支引用文件

```bash
cat .git/refs/heads/master
# 预期输出：一串 40 位哈希值（C2 的 SHA）

cat .git/refs/heads/feature
# 预期输出：另一串 40 位哈希值（C3 的 SHA）
```

### 1.3 HEAD 解析链路

```bash
# HEAD 指向谁？
git rev-parse HEAD
# 输出：当前提交的完整哈希

# HEAD 指向哪个分支？
git symbolic-ref HEAD
# 输出：refs/heads/master
```

### 1.4 切换分支观察 HEAD 变化

```bash
# 切换到 feature 分支
git checkout feature

# 观察 HEAD 文件
cat .git/HEAD
# 输出：ref: refs/heads/feature（变了！）

# HEAD 解析到的提交也变了
git rev-parse HEAD
# 输出：C3 的哈希（feature 分支的提交）
```

### 1.5 分离 HEAD 实验

```bash
# 切换回 master
git checkout master

# 用提交哈希切换（分离 HEAD）
git checkout $(cat .git/refs/heads/master~1)

# 观察 HEAD 文件
cat .git/HEAD
# 输出：一串哈希值（没有 ref: 前缀！直接指向提交）

# 确认处于分离状态
git branch
# 会看到：(HEAD detached at xxx)

# 恢复正常
git checkout master
```

---

## 实验二：验证 Git Hooks

> 钩子已通过 `core.hooksPath` 配置为 `.githooks/` 目录

### 2.1 测试 pre-commit 钩子

```bash
# 修改文件（加入调试代码）
echo 'console.log("debug");' >> index.js
git add .
git commit -m "Test pre-commit hook"

# 观察输出：会看到 pre-commit 钩子的检查信息
# 包括：调试代码扫描、大文件检查
```

### 2.2 测试 commit-msg 钩子

```bash
# 正常提交
git commit -m "normal commit"
# 观察输出：会显示提交信息内容

# 试试超长提交信息
git commit -m "这是一条非常非常非常非常非常非常非常非常非常非常非常非常长的提交信息"
# 观察输出：会警告超过 50 个字符
```

### 2.3 测试 post-checkout 钩子

```bash
# 切换分支
git checkout feature
# 观察输出：会显示 HEAD 变化信息

# 切换回 master
git checkout master
# 再次观察输出
```

### 2.4 测试 pre-push 钩子

```bash
# 尝试推送（需要有远程仓库）
git push origin master
# 观察输出：会检测到受保护分支并警告
```

### 2.5 跳过钩子

```bash
# 紧急情况可以跳过所有钩子
git commit --no-verify -m "emergency fix"
git push --no-verify
```

---

## 实验三：理解 refs/for/（Gerrit 场景）

```bash
# 普通推送（推送到 refs/heads/*）
git push origin master
# 等价于：git push origin master:refs/heads/master
# 代码直接进入远程 master 分支

# Gerrit 推送（推送到 refs/for/*）
git push origin master:refs/for/master
# 代码进入 Gerrit 审查队列，不会直接进入 master

# pre-push 钩子就是用来防止你误用第一种方式！
```

---

## 常用调试命令速查

```bash
# 查看 HEAD
cat .git/HEAD
git rev-parse HEAD
git symbolic-ref HEAD

# 查看所有引用
git show-ref
ls -R .git/refs/

# 查看提交对象
git cat-file -p HEAD          # 查看当前提交的内容
git cat-file -t HEAD          # 查看类型（commit）
git cat-file -p HEAD^{tree}   # 查看文件树

# 查看差异
git diff master feature       # 比较两个分支
git log --oneline --graph --all  # 图形化查看历史
```
