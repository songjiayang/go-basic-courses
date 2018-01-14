### 为什么选择 VSCode?

* 微软官方出品，更新迭代快
* 强大的插件：代码跳转，自动格式化，错误检测等

### 下载安装

* 登录 vscode 官网（https://code.visualstudio.com/）
* 根据操作系统选择对应包下载

  ![shell command](../pics/install-vscode.png)

### 将 `code` 命令添加到系统 PATH 中

> 效果：在终端输入 `code <filename/foldername>` 就能用 vscode 打开文件或文件夹

* 以 mac 为例：在 vscode 中使用快捷键 `command + shif + p`,
* 输入 shell command，选择 `Shell Command:Install 'code' command in PATH`，如下图：

  ![shell command](../pics/shell-command.png)

### 安装 Go 插件

* 登录 vscode 官网 `Extensions`模块：https://marketplace.visualstudio.com/VSCode
* 搜索 `go` 插件
* 推荐安装 `lukehoban` 的 go 插件： https://marketplace.visualstudio.com/items?itemName=lukehoban.Go
* 点击 `install` 后就打开 vscode 界面，进行安装

  ![shell command](../pics/go-extension.png)

### Go 开发相关配置

```
"files.autoSave": "onFocusChange",
"editor.formatOnSave": true,
"go.gopath":"${workspaceRoot}:/Users/jinxue/golib", // 当前工作空间${wordspaceRoot}加上系统 GOPATH 目录
"go.goroot": "/usr/local/Cellar/go/1.9/libexec", // go 的安装目录
"go.formatOnSave": true, //在保存代码时自动格式化代码
"go.formatTool": "goimports", //使用 goimports 工具进行代码格式化，或者使用 goreturns 和 gofmt
"go.buildOnSave": true, //在保存代码时自动编译代码
"go.lintOnSave": true, //在保存代码时自动检查代码可以优化的地方，并给出建议
"go.vetOnSave": false, //在保存代码时自动检查潜在的错误
"go.coverOnSave": false, //在保存代码时执行测试，并显示测试覆盖率
"go.useCodeSnippetsOnFunctionSuggest": true, //使用代码片段作为提示
"go.gocodeAutoBuild": false //代码自动编译构建
```
