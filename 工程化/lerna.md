# lerna              

2022-4-14 （记）

## 命令

### `lerna init`

创建一个lerna仓库，或将现有的仓库升级为lerna仓库

参数

-i / --independent 使用独立的版本控制模式

### `lerna bootstap`

执行引导历程（bootstrap）。会按照所需的依赖，及连接（本地）任何交叉依赖（可以通过import 或者require（）**直接通过软件**包的名字，引用项目中的其他包）

参数

--hoist 会将package目录下的公共模块抽离到最顶层

### `lerna import <pathtoRepo>`

将本地路径<pathtorepo>中的软件包导入packages/directory-name中并提交commit

### `lerna-publish`

为更新过的软件包创建一个新版本.提示输入新版本号并更新git和npm上的所有软件包

参数

--dist-tag [tagname] <latest|bata>  使用给定的npm dist-tag (默认为latest)

-c/--canary  创建一个canary版本

### `lerna-verson`

 标识自上一个有标记的版本以来已经更新的包。 提示输入新版本。 修改包package以反映新版本， 提交这些更改并标记提交。 推送到远程git。

--message <msg>  git 的commit信息

### `lerna changed`

检查上次发布依赖那些软件包被修改过

### `lerna diff [package?]`

列出所有或某个软件包自上次以来的修改情况

### `lerna run [script]`

在每个包含 [script] 脚本的软件包中运行此npm 脚本

### `lerna ls`

列出当前仓库中所有的公共软件包

### `lerna create <name> [loc?]`

创建一个新的被lerna管理的package

### 小贴士

使用 [yarn workspaces](https://link.juejin.cn/?target=https%3A%2F%2Fyarnpkg.com%2Flang%2Fzh-Hans%2Fdocs%2Fworkspaces%2F) 结合 Lerna `useWorkspaces` 可以实现 [Lerna Hoisting](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flerna%2Flerna%2Fblob%2Fmain%2Fdoc%2Fhoist.md)。这并不是多此一举，这可以让你在统一的地方（根目录）管理依赖，这即节省时间又节省空间。

配置lerna.json

```json
{
  npmclient: "yarn",
  useWorkspaces: "ture",
}
```

同时顶级package.json必须包含一个workspaces数组：

```json

{
  ...
  workspaces: ["packages/*"]
}
```

