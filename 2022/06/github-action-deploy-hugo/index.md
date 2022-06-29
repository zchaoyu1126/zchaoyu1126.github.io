# 使用Github Action自动部署Hugo


 使用Loveit主题的拓展short codes之后，由于在markdown之中引入了`{{}}`语法，使得Liquid在解析markdown中的代码块时出错，采用的解决方法是禁用Jekyll，因为hugo已经生成了静态站点所需的文件，不需要再使用Jekyll。在解决此问题的同时，学习于了解了Github上的Action功能，并使用Github Action完成了站点的自动部署。

<!--more--> 

## 0 前言

在未引入Github Action时，我部署在Github上的静态网站直接将Hugo生成的public目录直接上传至zchaoyu1126.github.io仓库中，然后自动触发pages build and deployment Action。等待一段时间后，就能通过访问zchaoyu1126.github.io查看效果。

但是，我在使用Loveit主题的拓展short codes之后，由于在markdown之中引入了`{{}}`语法，使得Liquid在解析markdown中的代码块时出错。

> 具体的原因，不是很清楚并没有深究。

{{< image src="img7.png" caption="Build with Jekyll 失败" src_s="img7.png" src_l="img7.png" >}}

{{< image src="img1.png" caption="Liquid syntax error" src_s="img1.png" src_l="img1.png" >}}

为了解决本问题我做了一些探索，百般曲折过后，还是在[官方文档](https://docs.github.com/cn/pages/getting-started-with-github-pages/about-github-pages#static-site-generators)中找到了解决办法。

{{< image src="img2.png" caption="官方文档中的说明" src_s="img2.png" src_l="img2.png" >}}

禁用Jekyll有效的原因：hugo已经为我们生成了静态站点所需的文件，不需要再使用Jekyll。

## 1 Hugo的使用

Hugo的基础使用：[https://www.gohugo.org/](https://www.gohugo.org/)

## 2 使用Github Action部署Hugo

关于Github Action，请参考[阮一峰老师的博客](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

以下内容参考：[https://tianhui.xin/blog/2019/11/17/hugousegithubactionstopages/](https://tianhui.xin/blog/2019/11/17/hugousegithubactionstopages/)

### 2.1 建立仓库

首先在github上建立两个仓库，

- zchaoyu1126/zchaoyu1126.github.io：public目录的远端仓库（静态网页部署站点）
- zchaoyu1126/blog：hugo工作目录的远端仓库

### 2.2 .gitignore文件

在hugo的工作目录中新建一个.gitignore文件，从而将public文件夹排除在git仓库管理范围外。.gitignore内容如下，

```
public/
```

### 2.3 添加公钥和私钥

**2.3.1** 打开命令行，输入如下命令会在当前文件夹下生成一组密钥

```bash
ssh-keygen -t ed25519 -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```

pub后缀为公钥，另一个为私钥

{{< image src="img3.png" caption="公钥与私钥" src_s="img3.png" src_l="img3.png" >}}



**2.3.2** 在zchaoyu1126/zchaoyu1126.github.io仓库中，点击Settings，添加Depoly keys

{{< image src="img4.png" caption="添加公钥" src_s="img4.png" src_l="img4.png" >}}



**2.3.3** 在zchaoyu1126/blog仓库中，点击Settings，添加Secrets

{{< image src="img5.png" caption="添加私钥" src_s="img5.png" src_l="img5.png" >}}

{{< admonition info "关于公钥密钥的解释">}}

当我们在本地写完markdown文件之后向远程仓库提交，也就是zchaoyu1126/blog，远程仓库会自动触发Github Action，这个Action会自动向zchaoyu1126/zchaoyu1126.github.io仓库部署静态网站。这二者之间的安全交互便是通过这一组密钥。

{{< /admonition >}}

### 2.4 禁用Jekyll

~~在zchaoyu1126/zchaoyu1126.github.io仓库中新建一个.nojekyll文件。其内容为空，表示禁用jekyll。~~

{{< image src="img6.png" caption="禁用Jekyll" src_s="img6.png" src_l="img6.png" >}}

{{< admonition info "血泪史" >}}

事实证明，这种方法不可行，2.5中的部署过程中向github写数据时，会把原有的.nojekyll文件删掉。

{{< /admonition >}}

### 2.5 添加Github Action

在zchaoyu1126/blog中新建一个Action，其内容如下：

```yaml
# ACTIONS_DEPLOY_KEY  根据自身情况进行修改，即2.3.3中输入的Name
# EXTERNAL_REPOSITORY 根据自身情况进行修改

name: GitHub Pages Deploy #自动化的名称
on:
  push: # push的时候触发
    branches: # 那些分支需要触发
      - master
jobs:
  build:
    runs-on: ubuntu-latest # 镜像市场
    steps:
      - name: checkout # 步骤的名称
        uses: actions/checkout@master # 软件市场的名称
        with: # 参数
          submodules: true
          
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.91.2'
          extended: true
          
      - name: Build
        run: hugo --minify
        
      - name: Forbid Jekyll            # 由于2.4节的方法失效，所以在这里生成.nojekyll文件
        run: touch ./public/.nojekyll
        
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v2.5.1
        env:
          ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          EXTERNAL_REPOSITORY: zchaoyu1126/zchaoyu1126.github.io
          PUBLISH_BRANCH: master
          PUBLISH_DIR: ./public
```

### 2.6 Git Push

创建完Action之后，会执行一次push并触发action。

将本地的hugo目录同步至远程仓库(zchaoyu1126/blog)后，访问zchaoyu1126.github.io即可看到效果。

## 3 参考资料

[https://www.gohugo.org/](https://www.gohugo.org/)

[https://hugoloveit.com/zh-cn/theme-documentation-extended-shortcodes/](https://hugoloveit.com/zh-cn/theme-documentation-extended-shortcodes/)

[https://tianhui.xin/blog/2019/11/17/hugousegithubactionstopages/](https://tianhui.xin/blog/2019/11/17/hugousegithubactionstopages/)

[https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

[https://docs.github.com/cn/pages/getting-started-with-github-pages/about-github-pages#static-site-generators](https://docs.github.com/cn/pages/getting-started-with-github-pages/about-github-pages#static-site-generators)
