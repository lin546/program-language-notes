# npm模块管理器

## 1、<font style="color:rgb(0, 0, 0);">切换镜像源</font>

<font style="color:rgb(0, 0, 0);">（1）全局修改 npm 的 registry</font>

```java
npm config set registry https://registry.npmmirror.com
```

<font style="color:rgb(51, 51, 51);">通过下面的命令确认是否切换成功：</font>

```java
npm config get registry
```

<font style="color:rgb(51, 51, 51);">如果你需要切换回 npm 的官方源，只需执行以下命令：</font>

```java
npm config set registry https://registry.npmjs.org
```

## <font style="color:rgb(34, 34, 34);">2、npm update</font>

<code><font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">npm update</font></code><font style="color:rgb(34, 34, 34);">命令可以更新本地安装的模块。</font>

```shell
# 升级当前项目的指定模块
$ npm update [package name]

# 升级全局安装的模块
$ npm update -global [package name]
```

<font style="color:rgb(34, 34, 34);">它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。</font>

<font style="color:rgb(34, 34, 34);">使用</font><code><font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">-S</font></code><font style="color:rgb(34, 34, 34);">或</font><code><font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">--save</font></code><font style="color:rgb(34, 34, 34);">参数，可以在安装的时候更新</font><code><font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">package.json</font></code><font style="color:rgb(34, 34, 34);">里面模块的版本号。</font>

```shell
// 更新之前的package.json
dependencies: {
  dep1: "^1.1.1"
}

// 更新之后的package.json
dependencies: {
  dep1: "^1.2.2"
}
```

<font style="color:rgb(34, 34, 34);">注意，从npm v2.6.1 开始，</font><code><font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">npm update</font></code><font style="color:rgb(34, 34, 34);">只更新顶层模块，而不更新依赖的依赖，以前版本是递归更新的。如果想取到老版本的效果，要使用下面的命令。</font>

```shell
npm --depth 9999 update
```


> 更新: 2026-01-17 14:19:41  
> 原文: <https://www.yuque.com/thinkspace/du51gc/xygatwwhrv0s5tyr>