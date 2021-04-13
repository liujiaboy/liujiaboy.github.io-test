---
title: 利用Github从零开始搭建Hexo博客，可迁移
date: 2020-03-06 22:07:06
tags:
    - Github
    - hexo

categories:
    杂记
---

## 准备

1. 安装Node.js，安装完成之后可以正常使用npm命令
2. 安装hexo，`npm install -g hexo-cli`

## 创建步奏

1. 创建git仓库，这里需要注意一下，GitPage会默认给分配二级域名，但是当仓库名与账号组织名称一样的情况下会省略掉二级域名，所以这里创建的名称为`{yourname}.github.io`，一定是以`github.io`结尾的。当然也可以创建其他的。只是在之后的配置上略有不同。

2. 创建仓库时使用readme进行初始化。

3. 同时新建一个hexo分支，并设置为默认分支。

4. 使用`hexo init {name}`来初始化，或者新建一个`{name}`文件夹，在该文件夹下，执行`hexo init`命令是一样的效果。

5. 依次执行

    `hexo clean`：clean
    `hexo g`：generate，编译生成HTML代码，放在根目录下的public下。
    `hexo s`：serve，提供serve命令把博客在本地运行起来，然后点击给出的链接就可以正常的使用了，如果链接打不开，可能是端口被占用，可以使用`hexo s -p 端口号`来指定端口号。

6. 我这里是用了一个比较主流的主题[next](https://github.com/theme-next/hexo-theme-next)，以此为例，大家喜欢什么主题自己配置就好。
    
    ```
    git clone https://github.com/theme-next/hexo-theme-next themes/next
    
    ```

    依次执行命令进行配置就好了。随后在根目录下的`_config.yml`文件里配置： `next`。
    

7. 删除主题下的git。这里有一点要注意的，在remote远程仓库之前，要把<b>themes文件夹</b>拉下来的主题，删除对应的<b>.git</b>文件删除掉。因为外部已经有了git，内部文件夹也有git时会出现问题。

8. 配置根目录下的`_config.yml`文件
    
    ```
    # Site
    title: 不会飞的小白
    subtitle: 'Stay Hungry, Stay Foolish'
    description: ''
    keywords: iOS, Swift, GitHub
    author: 不会飞的小白
    language: zh-CN
    timezone: ''
    ```
    <b>Site里头的东西，根据自身想法看着写就行。有一个需要注意的是language，它的值是根据对应的Theme中的语言相对应的。</b>

    ```
    # URL
    ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
    url: http://liujiaboy.github.io/
    root: /
    ```

    <b>这里也需要注意。如果添加了搜索功能或者没有按照{name}.github.io这种命名方式创建的，这里的是必须要填写的。URL：为Github Page生成后对应的地址。root：为哪一级为根目录。</b>
    
    ```
    # Extensions
    ## Plugins: https://hexo.io/plugins/
    ## Themes: https://hexo.io/themes/
    theme: next
    #主题
    
    # Deployment
    ## Docs: https://hexo.io/docs/deployment.html
    deploy:
      type: git
      repo: https://github.com/liujiaboy/liujiaboy.github.io.git
      branch: master
    
    ```
    <b>这里是对应的仓库地址，type一定是 git类型，分支可以选择，我这里是master。</b>
    
9. 上面的配置完成之后，再执行<b>步奏5</b>，本地预览没问题后，可以直接走发布。而发布也就是将生成的public文件中的内容push到deploy对应的仓库的分支中。
    
    `hexo d`:deploy: 发布。
    如果出现问题，执行`npm install hexo-deployer-git`安装git插件。
    
10. 这个时候，可以在根目录下，remote远程仓库了。
    
    `git remote add origin <远程版本库URL>`。这个时候本地默认就是之前创建的`hexo`分支。

    执行`git add .`, `git commit -m"xxx"`之后，在push之前，先把远程分支上的内容pull下来。这个时候执行
    
    `git pull origin hexo`
    
    如果出现问题，是因为本地有提交内容无法完成merge操作。将上面的执行命令改为
    
    `git pull origin hexo --allow-unrelated-histories`
    
    完成merge操作之后，执行`git push origin hexo`就可以了。
    
    
这个时候就已经完成了整体的库都放在Github上了。默认分支hexo存放所有的数据，master分支存放Github Page需要的数据。这就可以开心的玩耍了，也不用担心换设备迁移带来的问题了。


## hexo迁移

1. 确保当前设备已经有Node.js, hexo环境。
2. 从远程仓库clone到指定文件夹。
3. 在该文件夹下，执行
    
    ```
    npm install hexo --save
    ```
    这一步是必须执行的，创建hexo环境，如果在本电脑的环境下已经执行过下面两个命令，就不用重复执行了。
    ```
    npm install
    npm install hexo-deployer-git
    ```

    这个时候就没必要执行`hexo init`了，因为当文件夹下有内容的时候跟本执行不了，会报错。

4. 如果是多设备同时开发的话，那就需要用到 `git pull origin hexo`来拉取最新数据了，但是一般情况下不会出现，毕竟自己的博客不会涉及到多人协作。
5. 就可以开心的玩耍了。


## 相关配置

### NexT主题添加theme-next-canvas-nest几何动效

``` obj-c
cd themes/next
git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest
```
完成上诉操作之后，再next配置文件_config.yml中配置开启canvas_nest，如果没有相关配置，直接复制代码粘贴进去就行。

```
canvas_nest:
  enable: true
  onmobile: true # display on mobile or not
  color: '0,0,255' # RGB values, use ',' to separate
  opacity: 0.5 # the opacity of line: 0~1
  zIndex: -1 # z-index property of the background
  count: 99 # the number of lines
```

### 订阅 rss 

博客一般都需要rss订阅，如果要开启，需要安装一个插件。安装完成会自动生成相关配置。
    
```
npm install hexo-generator-feed --save
```
    
生成的rss对应的文件public/atom.xml

### 创建文章
```
hexo new <文章名>
```
新建一篇文章，使用markdown的格式。在一篇文章的开头，会有如下必要信息

```
---
title: 利用Github从零开始搭建Hexo博客，可迁移
date: 2020-03-06 22:07:06
tags:
    - Github
    - hexo

categories:
    - 杂记
---
```
其中title， date是必填的，tags就是标签，可选， categories：分类，也是可选。在next主题下，默认会生成目录。

### 创建标签页 tags
```
hexo new page tags
```

也会生成类似文章的md文件，可以在头部自行添加。

```
title: tags
date: 2020-03-06 22:07:06
type: tags
comments: false
```
type: 指定为tags
comments：是否允许评论

### 分类页 categories
```
hexo new page categories
```
指定type：categories

hexo默认会创建`archives`类型的文件，不会默认创建tags和categories，但是创建完成这两个之后，只需要指定type类型即可，不用在文件中添加其他数据。

### 搜索页
很多情况下需要搜索全站内容，所以有必要支持搜索。需要安装插件支持

```
npm install hexo-generator-searchdb --save
```

很多主题对应的搜索都不一样，但是搜索的插件都是必须的。这里需要修改上面提到的URL并指定root文件路径。

### 404页面
当页面出错的时候，就很有必要的添加一个404的错误页面。直接在根目录下创建一个404.md文件即可。也有会接入腾讯公益404广告的，自行google吧。

这个只针对绑定域名的有效，GitHub默认分配的是没有效果的。

```
---
title: 404 Not Found
date: 2020-03-06 11:06:55
---
<center>
    对不起，您访问的页面不存在或已被删除
</center>
```

### 创建about
```
hexo new page about
```

### 使用图片
1. 绝对路径引用
    当Hexo项目中只用到少量图片时，可以将图片统一放在source/images文件夹中，通过markdown语法访问它们。

    ```
    source/images/image.jpg
    // 这样就可以了
    ![](/images/image.jpg)
    ```

2. 相对路径引用
    图片除了可以放在统一的images文件夹中，还可以放在文章自己的目录中。文章的目录可以通过配置_config.yml来生成。
    ```
    post_asset_folder: true
    ```
    修改之后，执行命令`$ hexo new post_name`，在source/_posts中会生成文章post_name.md的同时，会生成文章同名的文件夹post_name。将图片资源放在post_name中，文章就可以使用相对路径引用图片资源了。
    ```
    ![](image.jpg)
    ```


## 总结
感谢大佬们做的总结，不一一列举了。感谢
[next: Github](https://github.com/theme-next/hexo-theme-next)


