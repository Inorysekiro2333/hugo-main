+++
title = '【持续更新】HUGO操作手册'
description = '记录一些HUGO的操作方法，随缘更新，想到哪写到哪'
date = '2026-01-13T11:22:27+08:00'
draft = false
image = 'PixPin_2026-01-17_20-16-24.jpg'
categories = ['HUGO','教程']
tags = ['hugo']
+++

## 如何创建新的文章

进到dev目录，打开cmd终端，输入以下指令：

```bash
hugo new content post/<新建文章的目录>/index.md
```

## 修改默认文章模板

在`archetypes\default.md`中修改即可。

## 创建定时推送到github的脚本
### 创建auto_sync.sh脚本
```bash
#!/bin/bash
# ============================================
# Hugo Blog 自动同步脚本
# 功能：自动拉取远程变更、提交本地更改、推送到GitHub
# ============================================

# 直接指定工作目录
cd /d/Develop/hugo-blog/dev

# 设置颜色输出
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 日志函数
log_info() {
    echo -e "${GREEN}[INFO]$(date '+%Y-%m-%d %H:%M:%S')${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]$(date '+%Y-%m-%d %H:%M:%S')${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]$(date '+%Y-%m-%d %H:%M:%S')${NC} $1"
}

# 检查是否有未提交的更改
check_changes() {
    if git diff --quiet && git diff --cached --quiet; then
        return 1
    else
        return 0
    fi
}

# 自动提交更改
auto_commit() {
    git add -A
    changed_files=$(git diff --cached --name-only)

    if [ -z "$changed_files" ]; then
        log_warn "没有需要提交的更改"
        return 1
    fi

    commit_msg="auto sync: $(date '+%Y-%m-%d %H:%M')"
    git commit -m "$commit_msg"

    if [ $? -eq 0 ]; then
        log_info "已提交更改: $commit_msg"
        return 0
    else
        log_error "提交失败"
        return 1
    fi
}

# 主流程
main() {
    log_info "========== 开始同步 =========="

    log_info "正在拉取远程变更..."
    git pull origin main

    if [ $? -ne 0 ]; then
        log_error "拉取失败，可能是合并冲突"
        exit 1
    fi

    if check_changes; then
        log_info "检测到本地有更改，正在提交..."
        if auto_commit; then
            log_info "正在推送到远程..."
            git push origin main
            if [ $? -eq 0 ]; then
                log_info "推送成功！"
            else
                log_error "推送失败"
                exit 1
            fi
        fi
    else
        log_warn "没有本地更改需要提交"
    fi

    log_info "========== 同步完成 =========="
}

main

```
### 创建定时任务
Windows操作系统使用`win + r -> taskschd.msc`
右键任务计划库->创建任务
填写：
- 名称：HugoBlogAutoSync
- 触发器：每日0点
- 操作：
    - 启动程序->程序：`D:\Develop\Git\usr\bin\bash.exe`，参数：`d:\Develop\hugo-blog\dev\auto_sync.sh`