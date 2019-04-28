[引用1](https://blog.csdn.net/lu_embedded/article/details/81100704 )  [引用2](https://www.jianshu.com/p/421cc442f06c)

​     GitBook 是一个基于 Node.js 的命令行工具，支持 Markdown 和 AsciiDoc 两种语法格式，可以输出 HTML、PDF、eBook 等格式的电子书。所以我更喜欢把 GitBook 定义为文档格式转换工具。 
 　所以，GitBook 不是 Markdown 编辑工具，也不是 Git 版本管理工具。市面上我们可以找到很多 Markdown 编辑器，比如 Typora、MacDown、Bear、MarkdownPad、MarkdownX、JetBrains’s IDE（需要安装插件）、Atom、简书、CSDN 以及 GitBook 自家的 GitBook Editor 等等。 
　简单来说，GitBook + Markdown + Git 带来的好处有：
* 语法简单

* 兼容性强

* 导出方便

* 专注内容

* 团队协作
  
   因为 GitBook 是基于 Node.js，所以我们首先需要安装 Node.js（ [下载地址](https://nodejs.org/en/download/)），找到对应平台的版本安装即可。

　　现在安装 Node.js 都会默认安装 npm（node 包管理工具），所以我们不用单独安装 npm，

* 打开命令行，执行以下命令安装 GitBook：
 `npm install -g gitbook-cli`

*  新建一个叫 mybook 的文件夹,在 mybook 文件夹下执行以下命令：
 `gitbook init`
执行完后，你会看到多了两个文件 —— README.md 和 SUMMARY.md，它们的作用如下：

 * README.md —— 书籍的介绍写在这个文件里

 * SUMMARY.md —— 书籍的目录结构在这里配置

* 编辑 SUMMARY.md 文件，内容修改为：
	
    ```
    # 目录
    * [前言](README.md)
    * [第一章](Chapter1/README.md)
      * [第1节：衣](Chapter1/衣.md)
      * [第2节：食](Chapter1/食.md)
      * [第3节：住](Chapter1/住.md)
      * [第4节：行](Chapter1/行.md)
    * [第二章](Chapter2/README.md)
    * [第三章](Chapter3/README.md)
    * [第四章](Chapter4/README.md)
    ```
* #### 目录结构

```
        .
        ├── book.json
        ├── README.md
        ├── SUMMARY.md
        ├── chapter-1/
        |   ├── README.md
        |   └── something.md
        └── chapter-2/
            ├── README.md
            └── something.md
```

* 回到命令行，在 mybook 文件夹中再次执行 gitbook init 命令。
   GitBook 会查找 SUMMARY.md 文件中描述的目录和文件，如果没有则会将其创建
   
* 执行 gitbook serve 来预览这本书籍
   执行命令后会对 Markdown 格式的文档进行转换，默认转换为 html 格式，最后提示 “Serving book on http://localhost:4000” 
   
  当你写得差不多，你可以执行 gitbook build 命令构建书籍，默认将生成的静态网站输出到 _book 目录。实际上，这一步也包含在 gitbook serve 里面，因为它们是 HTML，所以 GitBook 通过 Node.js 给你提供服务了。 
　　+ 当然，build 命令可以指定路径：
　　`gitbook build [书籍路径] [输出路径]`
　　+ serve 命令也可以指定端口：
　　`gitbook serve --port 2333`
　　+ 生成 PDF 格式的电子书
　　 `gitbook pdf ./ ./mybook.pdf`
　　+ 生成 epub 格式的电子书：
　　 `gitbook epub ./ ./mybook.epub`
　　+ 生成 mobi 格式的电子书：
　　 `gitbook mobi ./ ./mybook.mobi`
　　 如果生成不了，你可能还需要安装一些工具，比如 ebook-convert。或者在 Typora 中安装 Pandoc 进行导出。

　　除此之外，别忘了还可以用 Git 做版本管理呀！在 mybook 目录下执行 git init 初始化仓库，执行 git remote add 添加远程仓库（你得先在远端建好）。接着就可以愉快地 commit，push，pull

