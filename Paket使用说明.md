# Paket使用说明
## Questions

* 打包对组件版本化管理的影响
## 最后翻译完与Phoenix的规范对比一下
## Paket.dependencies
**paket.dependencies** 文件用来明确应用程序的依赖规则，该文件包含整个解决方案中所有项目的顶级依赖，而**paket.references**文件则明确了单独某个项目的依赖。    

为了给你一个概览，思考如下**paket.dependencies**文件：    

这个文件明确了Paket的Nuget依赖应该从nuget.org网站下载，Paket所有项目需要一下依赖：    
* NUnit version 2.6.3 <= x < 2.7 
* FAKE version 3.4 <= x < 4.0 
* DotNetZip with a version that is at least 1.9 
* FSUnit.fs from GitHub
* GitHub Gist number 1972349 
* External HTTP resource, i.e. 1n from FSSnip    

paket 利用上述的定义计算出具体的依赖解决方案，其中也包括间接依赖。然后产生的依赖关系图被放到**paket.lock**文件中    
ps:间接依赖通常在直接依赖的项目中的**paket.template**文件中定义，最后在直接依赖项目发布包的**nuspec**文件中体现。

如果只需列出直接依赖，可以使用**paket simplify**命令移除间接依赖    
### 包资源
Paket支持以下资源类型：    
### 全局选项
* 规定的Paket版本
* 严格的引用    
Paket通常在项目文件中添加项目各自的**paket.references**中列出的直接和间接依赖。但在严格模式下，Paket只会添加直接依赖，你需要自己添加间接依赖到**paket.references**。 （严格模式下不会下载间接依赖的包吗？放到项目文件夹的是什么东西） 
注意解决方案层不会受这个选项影响

## paket.references
**paket.references**文件用于明确仓库中的编译项目需要安装哪些依赖。Paket决定每个编译项目引用的依赖项存放在项目对应**paket.references**文件所在目录。    
这和NuGet的**package.config**文件作用很像，但又有一些重要区别：    
* 不用明确依赖包的版本；代替的依赖版本信息从**paket.lock**文件中获取。在最初的**paket install**或者之后的**paket update**命令执行过程中，版本信息依次从**paket.dependencies**文件定义的规则中获取。    
* 只有直接依赖应该被列出，除非使用严格引用。    
* 该文件只是一个普通的文本文件    

### 文件存放位置
Paket在**paket.dependencies**的下一层目录中寻找所有编译项目的**paket.references**文件。
### 布局
**paket.dependencies** 文件在规定的路径列出了来自**paket.lock**文件中任何项目需要引用的依赖
对于每一个带有**paket.references**文件的编译项目，使用**paket install** 和**paket update** 命令会添加**paket.references**文件中列出的依赖的引用，还包括他们的间接引用。    
## Paket install
该命令用来计算依赖图，下载依赖，更新项目。    
如果添加**--verbose**命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用**--log-file [path]**命令行参数，可以将日志信息存放到文件中。    
### 关注**paket.dependencies**文件的变化
自从上一次更新**paket.lock**文件，如果**paket.dependencies**发生变化（例如：添加了依赖项，版本限制发生改变），Paket会更新**paket.lock**文件，使之与**paket.dependencies**文件再次匹配。    
与**paket update**命令不同，**paket install**命令只会寻找**paket.dependencies**中改变依赖项的新版本，未改变的依赖项会使用**paket.lock**中的版本。
## paket update
将依赖更新到最新的版本    
如果添加**--verbose**命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用**--log-file [path]**命令行参数，可以将日志信息存放到文件中。
### 更新所有包
如果没有特殊指明某个包，所有**paket.dependencies**文件中的包都会更新。    
首先，会删除当前的**paket.lock**文件。然后，**Paket update** 命令会根据包方案算法重新计算当前的依赖方案，并将结果写到**paket.lock**文件中。接下来就开始下载依赖包并安放到项目中。    
如果你想保持当前**paket.lock**文件中的包依赖版本，请查看**paket install**命令。
### Updating a single group
### 更新**http**依赖
如果想更新一个文件，你需要使用带**--force**参数的**paket install**命令或**paket update**命令。    
为了减少强制安装依赖文件的数量，下载**http**依赖文件时使用groups会很有帮助。
## paket.template文件
**paket.template**文件用来明确使用**paket pack**命令创建**.nupkg**包的规则。