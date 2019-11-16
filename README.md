## VictorDan的个人博客源码
### 1、安装hexo
```shell
npm install -g hexo-cli
#初始化
hexo init
#generate生成命令
hexo g
#server命令启动服务
hexo s
```
### 2、安装主题
```shell
#安装yilia主题风格
git clone https://github.com/JoeyBling/hexo-theme-yilia-plus
#修改根目录的_config.yml(注意语法：冒号后面有空格)
theme: yilia
#在themes/yilia/_config.yml开启不蒜子统计字数
busuanzi:
 enable: true
```
### 3、首先这个yilia主题是确实模块的
```shell
npm i hexo-generator-json-content --save
```
#### 3.1、根目录的_config.yml，并不是themes文件夹，加上如下代码
```yaml
jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: false
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true
```
### 4、安装动态萌萌哒2d插件
```shell
#安装动态模型萌萌哒插件
npm install --save hexo-helper-live2d
```
#### 4.1、动态模型预览网站，选择你自己喜欢的
https://huaji8.top/post/live2d-plugin-2.0/
#### 4.2、根目录的_config.yml配置如下代码
```yaml
live2d:
  enable: true
  #enable: false
  scriptFrom: local # 默认
  pluginRootPath: live2dw/ # 插件在站点上的根目录(相对路径)
  pluginJsPath: lib/ # 脚本文件相对与插件根目录路径
  pluginModelPath: assets/ # 模型文件相对与插件根目录路径
  # scriptFrom: jsdelivr # jsdelivr CDN
  # scriptFrom: unpkg # unpkg CDN
  # scriptFrom: https://cdn.jsdelivr.net/npm/live2d-widget@3.x/lib/L2Dwidget.min.js # 你的自定义 url
  tagMode: false # 标签模式, 是否仅替换 live2d tag标签而非插入到所有页面中
  debug: false # 调试, 是否在控制台输出日志
  model:
    use: live2d-widget-model-shizuku
    # use: live2d-widget-model-wanko # npm-module package name
    # use: wanko # 博客根目录/live2d_models/ 下的目录名
    # use: ./wives/wanko # 相对于博客根目录的路径
    # use: https://cdn.jsdelivr.net/npm/live2d-widget-model-wanko@1.0.5/assets/wanko.model.json # 你的自定义 url
  display:
    position: right
    width: 145
    height: 315
  mobile:
    show: true # 是否在移动设备上显示
    scale: 0.5 # 移动设备上的缩放
  react:
    opacityDefault: 0.7
    opacityOnHover: 0.8
```
### 5、安装RSS插件
```shell
npm install hexo-generatgor-feed
```
#### 配置根目录的_config.yml文件
```yaml
# Extensions
plugins:
  hexo-generator-feed
#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20
```
#### 配置theme/yilia/_config.yml的配置
```yaml
rss: /atom.xml
```
### 添加文章的目录
#### themes/yilia/layout/_partial/post/toc.ejs的目录下新建toc.ejs
```javascript
<div id="toc" class="toc-article">
    <div class="toc-title">目录</div>
    <%- toc(item.content,{list_number: false})%>
</div>
```
#### themes/yilia/layout/_partial/article.ejs的目录下添加如下代码(可以放到最后)
```js
<!-- 目录内容 -->
        <% if (!index && post.toc){ %>
            <p class="show-toc-btn" id="show-toc-btn" onclick="showToc();" style="display:none">
            <span class="btn-bg"></span>
            <span class="btn-text">文章导航</span>
            </p>
            <div id="toc-article" class="toc-article">
                <span id="toc-close" class="toc-close" title="隐藏导航" onclick="showBtn();">×</span>
                <strong class="toc-title">文章目录</strong>
                <%- toc(post.content) %>
           </div>
           <script type="text/javascript">
            function showToc(){
                var toc_article = document.getElementById("toc-article");
                var show_toc_btn = document.getElementById("show-toc-btn");
                toc_article.setAttribute("style","display:block");
                show_toc_btn.setAttribute("style","display:none");
                };
            function showBtn(){
                var toc_article = document.getElementById("toc-article");
                var show_toc_btn = document.getElementById("show-toc-btn");
                toc_article.setAttribute("style","display:none");
                show_toc_btn.setAttribute("style","display:block");
                };
           </script>
        <% } %>     
        <!-- 目录内容结束 -->
```
## 发布到github静态网页的命令
```shell
hexo clean
hexo s -g
hexo d
```
# 访问地址：
https://victorblog.github.io/

