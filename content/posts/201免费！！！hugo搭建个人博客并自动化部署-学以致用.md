---
title: "201免费！！！hugo搭建个人博客并自动化部署"
date: 2021-01-20T22:37:29+08:00
draft: true
---

# 前言
本文详细介绍了“如何用hugo**免费**搭建个人博客并自动化发布部署”，原理为：hugo基于博客模板（皮肤）把markdown文件生成网页，利用github pages服务托管上述网页，实现免费博客。具体细节如下：
1. 将个人博客系统发布到github的特殊仓库的`blog`分支中；
2. 当提交本地仓库到github时，github通过webhook通知travis-ci；
3. travis-ci拉去最新的仓库，并执行特定脚本（hugo 生成页面脚本），生成html文件；
4. travis-ci自动将生成的html文件提交到本仓库的`master`分支；
5. 本仓库开启了github pages功能，可以通过特殊域名访问；

需要如下背景知识或条件：
- git基本操作
- github账户
- vscode编辑器

# 絮絮叨
很久很久之前，曾经用wordpress搭建过个人博客**极客工程师**。但忘记给服务器续费，站点被清空。沮丧、懊悔、无助，像是弄丢了心爱玩具的小孩。也因此很久没再弄个人博客，偶尔在公众号上写写随笔。

心血来潮，决定再弄一个博客，记录技术心得和成长感悟，同时以文会友获得更多机会。吸取教训，这次最核心的诉求是数据安全，其次是拓展性和美观性。虽然，也有公众号，但总感觉有诸多限制，不足以彰显个性凸显逼格。

# 需求分析

核心诉求如下：
- 安全性高：对于博客，文章是最主要的资产，一定要保证数据安全不容易丢失；
- 拓展性强：博客就是一个乐高玩具，一定要能自由定制功能，承载我的小创意；
- 写作方便：写博客主要时间都花费再写文章上，那么写作体验一定要好；
- 体验良好：本身是一名产品经理，产品体验肯定要考虑（交互设计及UI设计）；
- 成本要低：搭建博客本身就是为了“玩”，花太多金钱就不太值得了；

# 技术选型

目前市场上有很多博客搭建工具，主要分为两类：静态博客和动态博客。
- 动态博客：一套web系统，博文存入数据库，访问文章时从数据库读取博文，并基于模板渲染为html展示给用户。我之前用过的wordpress属于动态博客。
- 静态博客：通过工具，直接将博文编译成最终的html文件，放入服务器接口访问。当前用的最多的静态博客是hexo。

两种类型各有优缺点，前者功能强大可以集成功能模块，但需要专门的服务器部署各种运行环境及数据库；后者响应快，无需维护，成本还低，SEO友好。所以本次决定尝试静态博客，从静态博客的优秀代表hexo应该重点考虑，但发现很多人都吐槽hexo生成较慢，而且还需要安装nodejs才能使用，所以放弃了它。最终选了和它类似的hugo，主要原因如下：
- hugo相对于hexo不需要安装任何依赖，下载二进制包即可使用；
- hugo生成页面快，史称最快没有之一，方便本地调试；5秒生成6000个页面！
- hugo由golang开发，自己也一直在用golang，自带亲近感;

# 操作步骤（window10)

- 配置博客：安装hugo->新建站点->安装模板及预览->新增文章->配置站点
- 部署博客：新建仓库->授权travis->修改配置->提交站点->域名配置（可选）
## 配置博客

### 安装hugo

1. 下载hugo：点击[下载](https://github.com/gohugoio/hugo/releases/download/v0.80.0/hugo_extended_0.80.0_Windows-64bit.zip)`hugo.exe`，记得一定要下载扩展版，不然可能会报错
    - hugo有两种版本：一个是普通版，一个是扩展版（支持各种插件如sass）
    - 上述链接为hugo_extended_0.80.0_Windows-64bit,为扩展版
2. 安装hugo：将hugo.exe文件复制到任意一个文件夹（随心即可），同时在*系统环境变量*中设置hugo.exe所在目录，重启电脑（不重启可能不生效）
![20210122011409](https://raw.githubusercontent.com/AnsonCode/myblogtalk/main/img/20210122011409.png)

在命令行输入：
```bash
hugo version
```
若提示如下，说明安装成功。
```
Hugo Static Site Generator v0.80.0/extended windows/amd64 BuildDate: unknown
```
若提示如下，则说明*系统环境变量*设置错误或未生校。
```
hugo : 无法将“hugo”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼写，如果包括路径，请确保路径正确，然后再试一次。
所在位置 行:1 字符: 1
+ hugo version
+ ~~~~
    + CategoryInfo          : ObjectNotFound: (hugo:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

### 新建站点

1. 新建文件夹用于存储博客，用vscode打开文件夹
    - 点击 终端->新终端，打开终端
    - 在终端中，输入`cd ..`进入上一级目录
2. 在上一级目录中执行`hugo new site [替换成你的文件夹]`创建站点(提示如下)
```
Congratulations! Your new Hugo site is created in [你的目录].
Just a few more steps and you're ready to go:
1. Download a theme into the same-named folder. #下载主题到themes目录
   Choose a theme from https://themes.gohugo.io/ or #从官方下载主题
   create your own with the "hugo new theme <THEMENAME>" command. #新建主题
2. Perhaps you want to add some content. You can add single files 
   with "hugo new <SECTIONNAME>\<FILENAME>.<FORMAT>".# 新建内容
3. Start the built-in live server via "hugo server". # 本地启动服务
```
### 安装模板及预览
1. 前往hugo官网选择中意的模板下载，前往[这里](https://themes.gohugo.io/)
    - 官网提供了模板列表，真正下载一般要到github上下载
    - 这里我推荐一个不错的模板：[mogege](https://github.com/Mogeko/mogege)（本站在用）
2. 将下载的模板复制到themes目录即可[here](https://gohugo.io/news/0.48-relnotes/)
3. 执行如下指令后，访问`http://localhost:1313/`,即可在本地预览博客
```bash
hugo server --minify --theme mogege
```
### 新增文章
1. 在终端右上角点击“+”新建终端
2. 录入`hugo new posts/[文章名].md`新建文章
3. 在`content/posts`目录中可以看到该文章
```
--- #头部起始位
title: "001万物之始 内存对齐" #文章标题
date: 2021-01-21T01:12:44+08:00 #文章日期
draft: true #是否为草稿（false发布|true草稿）
--- #头部结束位

这里是文章内容，需要用markdown语法来写，其实很简单，等我填坑
```
4. 刷新`http://localhost:1313/posts`可以看到文章展示出来了

### 配置站点
1. 首先打开`config.toml`文件,各字段解释如下：
```toml
baseURL = "http://ansoncode.bazhentu.net/" # <head> 里面的 baseurl 信息，填你的博客地址
title = "Anson`s Blog" # 浏览器的标题
languageCode = "zh-cn" # 语言
hasCJKLanguage = true # 开启可以让「字数统计」统计汉字
theme = "mogege" # 主题 (需要自己下载)

paginate = 11 # 每页的文章数
enableEmoji = true # 支持 Emoji
enableRobotsTXT = true # 支持 robots.txt
googleAnalytics = "" # Google 统计 id

preserveTaxonomyNames = true

[blackfriday]
  hrefTargetBlank = true
  nofollowLinks = true
  noreferrerLinks = true

[Permalinks]
 posts = "/:year/:filename/"

[menu] #博客的菜单
  [[menu.main]] #第一个菜单
    name = "Blog"
    url = "/posts/" #菜单的访问路径
    weight = 1  #菜单的排序

  [[menu.main]]
    name = "Categories"
    url = "/categories/"
    weight = 2

  [[menu.main]]
    name = "Tags"
    url = "/tags/"
    weight = 3

  [[menu.main]]
    name = "About"
    url = "/about/"
    weight = 4

[params] #网站的参数
    since = 2017
    author = "Anson"                          # Author's name
    avatar = "https://i.loli.net/2021/01/20/PfjiWupdZrtzINV.png"           # Author's avatar
    subtitle = "Anson，会写代码的产品经理"    # Subtitle
    cdn_url = ""           # Base CDN URL
    home_mode = "" # post or other
    enableGitalk = false # gitalk 评论系统

    google_verification = ""

    description = "****" # (Meta) 描述
    keywords = "******" # site keywords 用于SEO优化

    beian = "xxx" #备案号
    baiduAnalytics = ""

    license= '本文采用<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/" target="_blank">知识共享署名-非商业性使用 4.0 国际许可协议</a>进行许可'

[params.social]
    GitHub = "AnsonCode"
    Twitter = "xxoo"
    Email   = "***@foxmail.com"
    Instagram = "xxoo"
    Wechat = "****"  # Wechat QRcode image
    Facebook = "xxoo"
    Telegram = "xxoo"
    Dribbble = "xxoo"
    Medium = "xxoo"

[params.gitalk] # Github: https://github.com/gitalk/gitalk （以后单独再讲解）
    clientID = "xxx" # Your client ID
    clientSecret = "xxx" # Your client secret
    repo = "xxx" # The repo to store comments
    owner = "xxx" # Your GitHub ID
    admin= "xxx" # Required. Github repository owner and collaborators. (Users who having write access to this repository)
    id= "location.pathname" # The unique id of the page.
    labels= "gitalk" # Github issue labels. If you used to use Gitment, you can change it
    perPage= 15 # Pagination size, with maximum 100.
    pagerDirection= "last" # Comment sorting direction, available values are 'last' and 'first'.
    createIssueManually= false # If it is 'false', it is auto to make a Github issue when the administrators login.
    distractionFreeMode= false # Enable hot key (cmd|ctrl + enter) submit comment.

```
2. 根据需要修改就行了，用不到的可以在前面加上`'#'`进行注释
3. 更多配置，如评论系统以后再说

## 部署博客
如果你有自己的服务器或者OSS，且不需要自动化部署，其实到这里就可以结束了。
但如果你希望自动化部署博客，那么follow me!

### 新建仓库
1. 前往github，新建仓库
    - 仓库名称为`[git用户名].github.io`(必须，这是一个特殊的仓库)
2. 设置仓库为public,且开启github pages服务
    - 在仓库的`Setting`->`GitHub Pages`中开启pages服务
    - 选择分支为`master`（需要等travis生效后再回来设置，不然没有master分支）
3. 执行以下命令，初始化仓库
```bash
git init
git remote add origin [你的仓库地址]
git checkout -b blog #新建blog分支并设置当前分支为blog
git add .
git commit -m "first commit"
git push -u origin blog #推送到github上
```
4. 在github的项目设置中把blog设置为主分支

### 授权travis
1. 前往[https://www.travis-ci.org/](https://www.travis-ci.org/)，并用github账户登陆
2. 前往[https://www.travis-ci.org/account/repositories](https://www.travis-ci.org/account/repositories)授权travis访问刚刚新建仓库的权限

![20210122030216](https://raw.githubusercontent.com/AnsonCode/myblogtalk/main/img/20210122030216.png)

3. 在travis当前仓库的后台设置`$GH_APIKEY2`(详情见`.travis.yml`）为github的token
    - 前往`https://github.com/settings/tokens`获取git token,权限为`write:packages`
    - $GH_APIKEY2对应的仓库设置为`blog`(原因是仅blog分支会触发脚本)
4. 在本地的博客文件夹根目录增加`.travis.yml`文件，文件内容如下
```yml
language: go

branches:
  only:
    - blog  # 设置自动化部署的源码分支(仅当前分支更新时会触发脚本)
    - 2-travis-auto-depoly # test for travis

# cache this directory
cache:
  directories:
    - themes
    # - public
    #- themes 主题没有更改时可以缓存

before_install:
- export TZ='Asia/Shanghai'  # 设置时区

# 安装依赖组件
install:
  - wget https://github.com/gohugoio/hugo/releases/download/v0.80.0/hugo_extended_0.80.0_Linux-64bit.tar.gz #下载hugo压缩包（扩展版本支持scss）
  - tar -xzvf hugo_extended_0.80.0_Linux-64bit.tar.gz #解压压缩包
  - chmod +x hugo   #授权hugo执行权限
  - export PATH=$PATH:$PWD
  - hugo version    #查看Hugo版本
  - git log -p -2 | cat  #查看本次提交变更（调试时用，可以注释）
  - rm -rf public/*      #删除blog仓库下的public文件

script:
  - hugo  --enableGitInfo  #执行hugo生成html到public目录下

# 构建完成后会自动更新Github Pages
deploy:
  provider: pages   #github pages部署
  skip-cleanup: true
  local-dir: public #部署的内容是刚刚生成的html站点
  target-branch: master #部署到当前仓库的master分支
  github-token: $GH_APIKEY2  #上传到该仓库需要push权限，这个是在travis后台设置的github token
  keep-history: true
  on:
    branch: blog
```

### 修改配置
1. 打开`config.toml`文件，修改参数
2. 将`baseURL`的值修改为`"[git用户名].github.io"`
3. 将`theme`的值修改为`"mogege"`(主要是为了匹配`.travis.yml`文件中的`hugo  --enableGitInfo`)

### 提交站点
1. 在本地初始化git，并新建blog分支
2. 将本地仓库映射到github仓库
3. 提交本地仓库的blog分支到github的blog分支
```bash
git add .
git commit -m "测试travis自动化部署"
git push -u origin blog #注意-》提交本地blog分支到blog分支
```

### 域名配置(可选)

自动化部署时github pages的custom domin被覆盖



# 参考
- hexo发布之后GitHub Pages自定义域名失效：https://craftboss.net/2019/10/22/hexo%E5%8F%91%E5%B8%83%E4%B9%8B%E5%90%8Egithubpage%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E5%A4%B1%E6%95%88/

- https://www.gohugo.org/doc/templates/functions/


- Hexo 博客终极玩法：云端写作，自动部署：https://segmentfault.com/a/1190000017797561?utm_source=tag-newest
- Hugo搭建个人博客与自动部署：https://kxcblog.com/post/blog/1.hugo-blog/
- Hugo 自动化部署脚本 deploy.py：https://www.gohugo.org/2015/11/21/hugo-deploy-script/
- 搭建 Hugo 静态博客并配置自动部署：https://www.hyec.me/posts/hugo-and-auto-deploy/
- 使用Travis-CI部署Hugo，实现自动化部署:https://blog.csdn.net/still_night/article/details/104838505/
- hugo模板配置：https://mogeko.me/2018/018/
- 搭建 Hugo 静态博客并配置自动部署：http://malonghua.com/post/25.html
- ⭐使用Travis CI自动构建Hexo静态博客：http://researchlab.github.io/2016/05/08/travis-ci-deploy-hexo-blog/
- Hugo 从入门到会用：https://olowolo.com/post/hugo-quick-start/?utm_source=cyhour.com