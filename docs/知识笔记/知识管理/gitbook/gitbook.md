# gitbook



GitBook 是一个 Node.js 环境下，用于构建电子书的工具。

感觉和gitpage 联合使用会很好用。



> 参考
>
> https://www.jianshu.com/p/3d03ab330df5



gitbook 要装 node js

gitbook 和 git 没啥关系，

gitbook是个管理markdown文件并可以构建发布的工具



对应成品 如果上传github 的话，要对应选中 ignore



参考文档

https://www.jianshu.com/p/421cc442f06c



里面的

```powershell
gitboot -v 
```

这句似乎有问题



```powershell
gitboot --verison
```







# gitbook-plugin

> 安装插件参考
>
> https://www.cnblogs.com/mingyue5826/p/10307051.html
>
> https://segmentfault.com/a/1190000019806829



装东西

```sh
## 中文搜索
npm install gitbook-plugin-search-pro

## mermaid
npm install gitbook-plugin-mermaid-gb3

## 折叠目录
npm install gitbook-plugin-chapter-fold
npm install gitbook-plugin-expandable-chapters
npm install gitbook-plugin-expandable-chapters-small

## 回到顶部
npm install gitbook-plugin-back-to-top-button

## 可调节侧边栏
npm install gitbook-plugin-splitter

```



添加到配置里

```sh
vim book.json

```

```json
{
    "plugins": ["splitter","-lunr","mermaid-gb3","chapter-fold","expandable-chapters","expandable-chapters-small","-search", "search-pro","back-to-top-button"],
    "pluginsConfig": {
        "disqus": {
                "shortName": "druler"
        }
  }
}
```





## gitbook-plugin-summury

用以自动生成 summary 的工具



http://self-publishing.ebookchain.org/3-%E5%A6%82%E4%BD%95%E6%89%93%E9%80%A0%E8%87%AA%E5%B7%B1%E7%9A%84%E5%B9%B3%E5%8F%B0%EF%BC%9F/2-Summary%E7%9A%84%E4%BD%BF%E7%94%A8.html







# gitbook本地启动

```sh
gitbook build
gitbook serve
```







