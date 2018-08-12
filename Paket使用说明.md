# Paket使用说明
## Questions
* 打包对组件版本化管理的影响
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

paket 利用上述的定义计算出具体的依赖解决方案，其中也包括间接依赖。然后产生的依赖图被放到**paket.lock**文件中    

如果只需列出直接依赖，可以使用**paket simplify**命令移除间接依赖    
### 包资源
Paket支持以下资源类型：    
### 全局选项
* 规定的Paket版本
* 严格的引用
Paket通常在项目文件中添加项目各自的**paket.references**中列出的直接和间接依赖。但在严格模式下，Paket只会添加直接依赖，你需要自己添加间接依赖到**paket.references**。 （严格模式下不会下载间接依赖的包吗？放到项目文件夹的是什么东西） 
注意解决方案层不会受这个选项影响

