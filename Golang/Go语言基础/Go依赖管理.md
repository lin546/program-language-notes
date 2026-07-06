# Go 依赖管理

## 一、Go mod 的使用
Go mod管理包的基本步骤:

1. 初始化项目：使用go mod init命令来初始化一个新的Go模块，生成一个go.mod文件，用于管理项目依赖。
2. 添加依赖：使用go get命令来添加一个新的依赖包到项目中。
3. 更新依赖：使用go get -u命令来更新依赖包到最新版本。
4. 删除依赖：使用go mod tidy命令来删除项目中没有用到的依赖包。
5. 查看依赖：使用go list命令来查看当前项目的依赖包



> 更新: 2023-08-13 01:38:22  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/gbpubamx6irvkq2e>