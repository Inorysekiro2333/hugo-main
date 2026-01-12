+++
date = '2026-01-11T21:44:13+08:00'
draft = true
title = '[Stack美化]一些样式修改'
+++

## 修改字体

（1）首先下载字体文件

（2）把字体放在 `assets/font`下，自己创建文件夹

![字体路径](font.jpg)

（3）将以下代码修改并复制到`layouts/partials/footer/custom.html`文件中(文件不存在就自己创建)

- **字体名**：给字体命名一个别名，随便填写就好，保持统一就行
- **字体文件名**：字体文件的全名，带后缀名的，也就是 **xxx.ttf**

```html
<style>
  @font-face {
    font-family: '字体名';
    src: url({{ (resources.Get "font/字体文件名").Permalink }}) format('truetype');
  }

  :root {
    --base-font-family: '字体名';
    --code-font-family: '字体名';
  }
</style>
```

## 添加最后更新时间

（1）在`hugo.yaml`中加入以下配置

```yaml
# 更新时间：优先读取git时间 -> git时间不存在，就读取本地文件修改时间
frontmatter:
  lastmod:
    - :git
    - :fileModTime

# 允许获取Git信息        
enableGitInfo: true
```

（2）修改github action文件 `.github/workflows/xxx.yaml`， 在运行 `hugo -D`命令的step前加入以下配置

```yaml
name: deploy

# 代码提交到main分支时触发github action
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
        - name: Checkout
          uses: actions/checkout@v4
          with:
              fetch-depth: 0
        # 获取时间      
        - name: Git Configuration
          run: |
            git config --global core.quotePath false
            git config --global core.autocrlf false
            git config --global core.safecrlf true
            git config --global core.ignorecase false 

        - name: Setup Hugo
          uses: peaceiris/actions-hugo@v3
          with:
              hugo-version: "latest"
              extended: true

        - name: Build Web
          run: hugo -D

        - name: Deploy Web
          uses: peaceiris/actions-gh-pages@v4
          with:
              PERSONAL_TOKEN: ${{ secrets.TOKEN }}
              EXTERNAL_REPOSITORY: Inorysekiro2333/Inorysekiro2333.github.io
              PUBLISH_BRANCH: main
              PUBLISH_DIR: ./public
              commit_message: auto deploy
```

## 新增友情链接、归档双栏显示

修改`assets/scss/custom.scss` 文件（不存在则自行创建）， 引入以下css样式代码

```css
@media (min-width: 1024px) {
  .article-list--compact {
    display: grid;
    // 目前是两列，如需三列，则后面再加一个1fr，以此类推
    grid-template-columns: 1fr 1fr;
    background: none;
    box-shadow: none;
    gap: 1rem;

    article {
      background: var(--card-background);
      border: none;
      box-shadow: var(--shadow-l2);
      margin-bottom: 8px;
      margin-right: 8px;
      border-radius: 16px;
    }
  }
}
```

## 文章目录折叠&展开

将以下代码复制到`layouts/partials/footer/custom.html`文件中（不存在则自行创建）

```html
<style>
    #TableOfContents > ul, ol {
        ul, ol {
            display: none;
        }
        .open {
            display: block;
        }
    }
</style>

<script>
    function initTocHide() {
        // 判断是否存在文章目录
        let toc = document.querySelector(".widget--toc");
        if (!toc) {
            return;
        }
        // 监听滚动
        window.addEventListener('scroll', function() {
            //清除class值
            let openUl = document.querySelectorAll(".open");
            if (openUl.length > 0) {
              openUl.forEach((ul) => {
                ul.classList.remove("open")
              })
            }
            // 获取active-class
            let currentLi = document.querySelector(".active-class");
            if (!currentLi) {
                return
            }
            // 展示子ul
            if (currentLi.children.length > 1) {
                currentLi.children[1].classList.add("open")
            }
            // 展示父ul
            let ul = currentLi.parentElement;
            do {
                ul.classList.add("open");
                ul = ul.parentElement.parentElement
            } while (ul !== undefined && (ul.localName === 'ul' || ul.localName === 'ol'))
        });
    }
    initTocHide()
</script>
```
