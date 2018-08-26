# Paket使用说明
## Questions

* 打大包对组件版本化管理的影响
* `paket restore` 代替 `paket clear-cache`
* paket.references的作用，严格引用有什么用
* 关于paket install 和 paket update的区别，paket update会重新寻找最新的，如果缓存中没有，应该会先下到缓存中，如果缓存中有了，就应该不会下载了，所以对重复上传同样版本号的包没作用。
* paket pack传--version参数，这个版本号有啥用？
* paket pack命令中说产生.nuspec文件中的依赖要么采用paket.dependencies文件中，要么采用paket.lock文件中，那之前一直以为的使用paket.template文件中的？究竟是哪一种？   
实际试验表示采用paket.template文件中的。
## 最后翻译完与Phoenix的规范对比一下
## Paket.dependencies（完成）
**paket.dependencies** 文件用来明确应用程序的依赖规则，该文件包含整个解决方案中所有项目的顶级依赖，而**paket.references**文件则明确了单独某个项目的依赖。    

为了给你一个概览，思考如下**paket.dependencies**文件：  

     1: source https://api.nuget.org/v3/index.json   
     2:    
     3: // NuGet packages    
     4: nuget NUnit ~> 2.6.3   
     5: nuget FAKE ~> 3.4    
     6: nuget DotNetZip >= 1.9    
     7:     
     8: // Files from GitHub repositories.    
     9: github forki/FsUnit FsUnit.fs`   
    10:     
    11: // Gist files.        
    12: gist Thorium/1972349 timestamp.fs     
    13:       
    14: // HTTP resources.        
    15: http http://www.fssnip.net/1n decrypt.fs      

   

这个文件明确了Paket的Nuget依赖应该从nuget.org网站下载，Paket所有项目需要一下依赖：    
* NUnit version 2.6.3 <= x < 2.7 
* FAKE version 3.4 <= x < 4.0 
* DotNetZip with a version that is at least 1.9 
* FSUnit.fs from GitHub
* GitHub Gist number 1972349 
* External HTTP resource, i.e. 1n from FSSnip    

`paket` 利用上述的定义计算出具体的依赖解决方案，其中也包括间接依赖。然后产生的依赖关系图被放到**paket.lock**文件中    
ps:间接依赖通常在直接依赖的项目中的**paket.template**文件中定义，最后在直接依赖项目发布包的**nuspec**文件中体现。

如果只需列出直接依赖，可以使用**paket simplify**命令移除间接依赖    
### 包资源
Paket支持以下资源类型：    
* NuGet
* .NET CLI Tools
* Git
* GitHub and Gist
* HTTP (any single file from any site without version control) 

### 全局选项
* 规定的Paket版本    
对于一个**paket.dependencies**文件，可能需要一个特定的Paket版本。这个需求可以由以下方式实现，以**version**开头的一行，其后跟着所需**paket.exe**的版本号，还有可选的**bootstrapper**命令行参数。示例如下所示：

        1: version 3.24.1     
        2:     
        3: source https://api.nuget.org/v3/index.json     
        4: nuget FAKE         
        5: nuget FSharp.Core ~> 4      

    或者    

        1: version 3.24.1 --prefer-nuget    
        2:        
        3: source https://api.nuget.org/v3/index.json     
        4: nuget FAKE     
        5: nuget FSharp.Core ~> 4     
 
* 严格的引用    
Paket通常在项目文件中添加项目各自的**paket.references**中列出的直接和间接依赖。但在严格模式下，Paket只会添加直接依赖，你需要自己添加间接依赖到**paket.references**。 （严格模式下不会下载间接依赖的包吗？放到项目文件夹的是什么东西） 

        1: references: strict     
        2: source https://nuget.org/api/v2       
        3:        
        4: nuget Newtonsoft.Json ~> 6.0       
        5: nuget UnionArgParser ~> 0.7        

 
注意解决方案层不会受这个选项影响，Paket仍然会解析，锁定，下载所有间接引用。
* 预发布版本
如果你想依赖预发布版本，Paket能够协助你。与NuGet不同，Paket允许依赖不同的预发布通道：

        1: nuget Example >= 1.2.3 alpha  // At least 1.2.3 including alpha versions    
        2: nuget Example >= 2 beta rc    // At least 2.0 including rc and beta versions          
        3: nuget Example >= 3 rc         // At least 3.0 including rc versions    
        4：nuget Example >= 3 prerelease / /At least 3.0 including all prerelease versions


## paket.references(完成，具体现象没出来，但在输出日志中起作用了)
**paket.references**文件用于明确仓库中的编译项目需要安装哪些依赖。Paket决定每个编译项目引用的依赖项存放在项目对应**paket.references**文件所在目录。    
这和NuGet的**package.config**文件作用很像，但又有一些重要区别：    
* 不用明确依赖包的版本；代替的依赖版本信息从**paket.lock**文件中获取。在最初的**paket install**或者之后的**paket update**命令执行过程中，版本信息依次从**paket.dependencies**文件定义的规则中获取。    
* 只有直接依赖应该被列出，除非使用严格引用。 (ps:在非严格引用模式下，Paket会自动添加间接引用)   
* 该文件只是一个普通的文本文件    

### 文件存放位置
Paket在**paket.dependencies**的下一层目录中寻找所有编译项目的**paket.references**文件。
### 布局
**paket.references** 文件在规定的路径列出了来自**paket.lock**文件中任何项目需要引用的依赖    

    1：Newtonsoft.Json        
    2：UnionArgParser     
    3：DotNetZip      
    4：RestSharp      
    5：       
    6：group Test     
    7： NUnit     

 
对于每一个带有**paket.references**文件的编译项目，使用**paket install** 和**paket update** 命令会添加**paket.references**文件中列出的依赖的引用，还包括他们的间接引用。    
## Paket install(完成)
该命令用来计算依赖图，下载依赖，更新项目。    
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`log-file [path]`命令行参数，可以将日志信息存放到文件中。    
### 关注**paket.dependencies**文件的变化
自从上一次更新**paket.lock**文件，如果**paket.dependencies**发生变化（例如：添加了依赖项，版本限制发生改变），Paket会更新**paket.lock**文件，使之与**paket.dependencies**文件再次匹配。    
与**paket update**命令不同，**paket install**命令只会寻找**paket.dependencies**中改变依赖项的新版本，未改变的依赖项会使用**paket.lock**中的版本。
## paket update（完成）
将依赖更新到最新的版本    
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`--log-file [path]`命令行参数，可以将日志信息存放到文件中。
### 更新所有包
如果没有特殊指明某个包，所有**paket.dependencies**文件中的包都会更新。    
首先，会删除当前的**paket.lock**文件。然后，**Paket update** 命令会根据包方案算法重新计算当前的依赖方案，并将结果写到**paket.lock**文件中。接下来就开始下载依赖包并安放到项目中。    
如果你想保持当前**paket.lock**文件中的包依赖版本，请查看**paket install**命令。
### **Updating a single group**
### 更新**http**依赖
如果想更新一个文件，你需要使用带`--force`参数的**paket install**命令或**paket update**命令。    
为了减少强制安装依赖文件的数量，下载**http**依赖文件时使用groups会很有帮助。
## paket.template文件(未完成)
**paket.template**文件用来明确使用**paket pack**命令创建**.nupkg**包的规则。    
**tpye**说明符必须在template文件的第一行，他有两个可能的值：
- **file**:所有生产**.nupkg**包的的信息需要包含在template文件中
- **project**:Paket 会寻找匹配的项目文件，然后从项目中推到依赖和元数据
匹配的项目和template文件的必须在同一目录下。如果目录下只有一个项目，template文件可以命名为**paket.template**，否则template文件必须以以项目文件名+**.paket.template**命名。    
例如：    
    `Paket.Project.fsproj`    
    `Paket.Project.fsproj.paket.template`    
### 例子
#### 例1
使用**type project**的**paket.template**文件可能如下所示：

    1: type project   
    2: licenseUrl http://opensource.org/licenses/MIT    
用这个template创建一个**.nupkg**包文件    
* 包的名字为**Test.Paket.Package.[Version].nupkg**,
- **Version, Author和Description**从程序集特性中获取，
- 在包的**lib**目录下包含**d$(OutDir)\\$(ProjectName).* **路径下的所有文件(项目输出目录的所有文件)
* 引用所有被项目引用的包，
* 包括包引用，
* 包括对解决方案下其他项目，要求这个项目拥有一个**paket.template**文件    

 #### 例2
 使用**type file**的**paket.template**文件可能如下所示：

    1: type file    
    2：id Test.Paket.Package     
    3: version 1.0    
    4: authors Michael Newton    
    5: description   
    6:   description of this test package    
    7: files    
    8:   src/Test.Paket.Package/bin/Debug ==> lib    
 这个template文件会创建一个名为**Test.Paket.Package.\<version>.nupkg**的包文件，包文件的**lib**路径下包含**src/Test.Paket.Package/bin/Debug**目录下的内容。

 ### 通用元数据
 通用元数据可以以两种方式书写；要么在单独行上以属性名为前缀（属性名不区分大小写），要么在只有属性名的行后跟随一个缩进块。    
 例如：  

    1: description This is a valid Description    
    2:     
    3: DESCRIPTION     
    4:   So is this     
    5:   description     
    6:     
    7: description This would     
    8:   cause an error

创建一个`.nupkg`包，template文件中必须包含4个字段，以项目为基础的template文件可以忽略，但会以以下方式推断出来：
- **id**：**.nupkg**包的ID(也决定了包的文件名)。如果在以项目为基础的template文件中被忽略了，包的文件名会以反射得到的程序及名字命名。
- **version**：**.nupkg**包的版本。如果在以项目为基础的template文件中被忽略了，可以通过反射获取**AssemblyInformationalVersionAttribute**值，**or if that is missing the AssemblyVersionAttribute**.
- **authors**：**.nupkg**包的多个作者可以通过逗号分开，如果在以项目为基础的template文件中被忽略了，可以从**AssemblyCompanyAttribute**值中推断出来。
- **description**：作为`.nupkgg`包的介绍呈现。如果没说明，可以从**AssemblyDescriptionAttribute**值中推断出来。    
其他通用元数据属性都是可选的，直接和`.nupkg`包中`.nuspec`文件中的字段对应。
- **title**：
- **Owners**：   
- `...未完待续   `
#### 依赖及要打包的文件
#### 要打包的文件
#### 引用的项目
利用**include-referenced-projects**命令行参数，可以告诉Paket在打包时是否需要将引用的项目包含到包中。
#### 框架程序集引用
#### 依赖
依赖块如下所示：    

    1:dependencies 
    2:  FSharp.Core >= 4.3.1 
    3:  Other.Dep ~> 2.5 
    4:  Any.Version 
明确依赖范围的语法和`paket.dependencies`文件中的范围语法是相同的。    
可以用`CURRENTVERSION`充当依赖包当前版本的占位符：

    1:dependencies 
    2:  FSharp.Core >= 4.3.1 
    3:  Other.Dep ~> CURRENTVERSION 
`LOCKEDVERSION`占位符允许引用当前`paket.lock`文件中使用的依赖版本


#### PDB文件
通过**include-pdbs**命令行参数，可以告诉Paket是否需要将PDBs打包到包中。
`1: paket pack --include-pdbs true`    
这只有在使用以项目为基础的**paket.template**时才起作用。

### 注解
以**#** 或 **//**开头的行会在解析时会被当作注释并忽略。行结尾的注释只允许在所有的依赖限制行使用。

## paket pack（完成）
根据paket.template文件创建NuGet包 

    paket pack [--help] [--build-config <configuration>] [--build-platform <platform>]        
              [--version <version>] [--template <path>] [--exclude <package ID>]      
              [--specific-version <package ID> <version>] [--release-notes <text>]        
              [--lock-dependencies] [--minimum-from-lock-file] [--pin-project-references] [--symbols]     
              [--include-referenced-projects] [--project-url <URL>] <path>`        
                  
    UTPUT:        
          
       <path>                output directory for .nupkg files        
          
    PTIONS:     
          
       --build-config <configuration>     
                             build configuration that should be packaged (default: Release)       
       --build-platform <platform>        
                             build platform that should be packaged (default: check all known platform        
                             targets)     
       --version <version>   version of the package      
       --template <path>     pack a single paket.template file        
       --exclude <package ID>     
                             exclude paket.template file by package ID; may be repeated      
       --specific-version <package ID> <version>      
                             version number to use for package ID; may be repeated        
       --release-notes <text>     
                             release notes        
       --lock-dependencies   use version constraints from paket.lock instead of paket.dependencies        
       --minimum-from-lock-file       
                             use version constraints from paket.lock instead of paket.dependencies and        
                             add them as a minimum version; --lock-dependencies overrides this option     
       --pin-project-references       
                             pin dependencies generated from project references to exact versions (=)     
                             instead of using minimum versions (>=); with --lock-dependencies project     
                             references will be pinned even if this option is not specified       
       --symbols             create symbol and source packages in addition to library and content     
                             packages     
       --include-referenced-projects      
                             include symbols and source from referenced projects      
       --project-url <URL>   homepage URL for the package     
       --silent, -s          suppress console output      
       --verbose, -v         print detailed information to the console        
       --log-file <path>     print output to a file       
       --help                display this list of options.        
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`--log-file [path]`命令行参数，可以将日志信息存放到文件中。
### 创建NuGet包   
考虑以下**paket.dependencies**文件

    1: source https://nuget.org/api/v2           
    2:      
    3: nuget Castle.Windsor ~> 3.2       
    4: nuget NUnit    
其中一个项目有一个**paket.references**文件

    1： Castle.Windsor
当你运行**paket install**，你的**paket.lock**文件会如下所示：

    1: NUGET
    2:   remote: https://nuget.org/api/v2
    3:     Castle.Core (3.3.3)
    4:     Castle.Windsor (3.3.0)
    5:       Castle.Core (>= 3.3.0)
    6:     NUnit (2.6.4)
当你完成编码，期望为项目创建一个NuGet包，新建一个带`type project`的`paket.template`文件，然后运行一下命令：

    1：paket pack nugets --version 1.0.0
你也可以运行以下命令：

    1：paket pack nugets --version 1.0.0 --lock-dependencies
运行的命令不同，Paket会创建包的不同版本依赖，这些版本依赖用于创建包的`.nuspec`文件：    
|Dependency|Default|With locked dependencies
|-----|----|-----|
|Castle.Windsor|[3.2,4.0)|[3.3.0]

第一种命令（没有带`--lock-dependencies`参数）根据`paket.dependencies`文件中明确的内容创建版本依赖。第二种命令采用当前`paket.lock`文件中列出的解析版本，将版本依赖锁定到了特定的某个版本。


## paket push
 上传一个Nuget包

     1: paket push [--help] [--url <url>] [--api-key <key>] [--endpoint <path>] <p ath>
     2: 
     3: PACKAGE:
     4: 
     5:     <path>                path to the .nupkg file
     6: 
     7: OPTIONS:
     8: 
     9:     --url <url>           URL of the NuGet feed
    10:     --api-key <key>       API key for the URL (default: value of the NUG ET_KEY environment   variable)
    11:     --endpoint <path>     API endpoint to push to (default: /ap i/v2/package)
    12:     --silent, -s          suppress console output
    13:     --verbose, -v         print detailed information to the console
    14:     --log-file <path>     print output to a file
    15:     --help                display this list of options.
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`--log-file [path]`命令行参数，可以将日志信息存放到文件中。

## paket clear-cache
清除Nuget和git的缓存目录   

     1: paket clear-cache [--help] [--clearlocal]
     2: 
     3: OPTIONS:
     4: 
     5:     --clearlocal, --clear-local
     6:                           Clears local packages folder and   paket-files.
     7:     --silent, -s          suppress console output
     8:     --verbose, -v         print detailed information to the  console
     9:     --log-file <path>     print output to a file
    10:     --help                display this list of options.
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`--log-file [path]`命令行参数，可以将日志信息存放到文件中。

## Paket paket.bootstrapper.exe
bootstrapper下载最新稳定的**paket.exe**.默认地，bootstrapper为当前用户所有的项目缓存下载的各版本**paket.exe**。如果要求的版本不在缓存中，就会从**github.com**中下载。如果从GitHub中下载失败，就会尝试从**nuget.org**中下载该版本。    
当Nuget缓存目录被清除，缓存的各版本**paket.exe**也会被移除。    
Ctrl+C会终止下载进程。when fresh copy of paket.exe was found,bootstrapper会返回0，接下来的脚本就能能够继续执行。

    1: paket.bootstrapper.exe [prerelease|<version>] [--prefer-nuget] [--self] [-s] [-f]

 ### 选项

 ## paket add
 添加一个新的依赖

     1: paket add [--help] [--version <version constraint>] [--project <path>] [--group <name>]
     2:           [--create-new-binding-files] [--force] [--interactive] [--redirects] [--clean-redirects]
     3:           [--no-install] [--no-resolve] [--keep-major] [--keep-minor] [--keep-patch]
     4:           [--touch-affected-refs] [--type <packageType>] <package ID>
     5: 
     6: NUGET:
     7: 
     8:     <package ID>          NuGet package ID
     9: 
    10: OPTIONS:
    11: 
    12:     --version, -V <version constraint>
    13:                           dependency version constraint
    14:     --project, -p <path>  add the dependency to a single project only
    15:     --group, -g <name>    add the dependency to a group (default: Main group)
    16:     --create-new-binding-files
    17:                           create binding redirect files if needed
    18:     --force, -f           force download and reinstallation of all dependencies
    19:     --interactive, -i     ask for every project whether to add the dependency
    20:     --redirects           create binding redirects
    21:     --clean-redirects     remove binding redirects that were not created by Paket
    22:     --no-install          do not modify projects
    23:     --no-resolve          do not resolve
    24:     --keep-major          only allow updates that preserve the major version
    25:     --keep-minor          only allow updates that preserve the minor version
    26:     --keep-patch          only allow updates that preserve the patch version
    27:     --touch-affected-refs touch project files referencing affected dependencies to help incremental
    28:                           build tools detecting the change
    29:     --type, -t <packageType>
    30:                           the type of dependency: nuget|clitool (default: nuget)
    31:     --silent, -s          suppress console output
    32:     --verbose, -v         print detailed information to the console
    33:     --log-file <path>     print output to a file
    34:     --help                display this list of options.
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`--log-file [path]`命令行参数，可以将日志信息存放到文件中。

 
 ### 添加到一个项目
 默认地，packages只会被添加到解决方案路径，不会被添加到任何项目路径。但将包添加到一个特定的项目也是能做到的：

    1：paket add <package ID> --project <project>

### 例子
考虑以下**paket.dependencies**文件：

    1：source https:/nuget.org/api/v2
    2：
    3：nuget FAKE
现在我们运行**paket add NUnit --version '~> 2.6' --interactive**命令来安装包：

     1: $ paket add NUnit --version '~> 2.6' --interactive
     2: Paket version 5.0.0
     3: Adding NUnit ~> 2.6 to ~/Example/paket.dependencies into group  Main
     4: Resolving packages for group Main:
     5:  - NUnit 2.6.4
     6:  - FAKE 4.61.3
     7: Locked version resolution written to ~/Example/paket.lock
     8: Dependencies files saved to ~/Example/paket.dependencies
     9:   Install to ~/Example/src/Foo/Foo.fsproj into group Main?
    10:     [Y]es/[N]o => y
    11: 
    12: Adding package NUnit to ~/Example/src/Foo/paket.references into     gro up Main
    13: References file saved to ~/Example/src/Foo/paket.references
    14:   Install to ~/Example/src/Bar/Bar.fsproj into group Main?
    15:     [Y]es/[N]o => n
    16: 
    17: Performance:
    18:  - Resolver: 12 seconds (1 runs)
    19:     - Runtime: 214 milliseconds
    20:     - Blocked (retrieving package details): 86 milliseconds (4  tim es)
    21:     - Blocked (retrieving package versions): 3 seconds (4 times)
    22:     - Not Blocked (retrieving package versions): 6 times
    23:     - Not Blocked (retrieving package details): 2 times
    24:  - Disk IO: 786 milliseconds
    25:  - Average Request Time: 1 second
    26:  - Number of Requests: 12
    27:  - Runtime: 14 seconds

安装操作会将包依赖添加到所选的**paket.references**文件和**paket.dependencies**文件中。注意命令中的版本限制被保存了。

    1: source https:/nuget.org/api/v2
    2: 
    3: nuget FAKE
    4: nuget NUnit ~> 2.6

## paket restore
下载已经计算好的依赖图

     1: paket restore [--help] [--force] [--only-referenced] [--touch-affected-refs] [--ignore-checks]
     2:               [--fail-on-checks] [--group <name>] [--project <path>] [--references-file <path>]
     3:               [--target-framework <framework>]
     4: 
     5: OPTIONS:
     6: 
     7:     --force, -f           force download and reinstallation of all dependencies
     8:     --only-referenced     only restore packages that are referenced by paket.references files
     9:     --touch-affected-refs touch project files referencing affected dependencies to help incremental
    10:                           build tools detecting the change
    11:     --ignore-checks       do not check if paket.dependencies and paket.lock are in sync
    12:     --fail-on-checks      abort if any checks fail
    13:     --group, -g <name>    restore dependencies of a single group
    14:     --project, -p <path>  restore dependencies of a single project
    15:     --references-file <path>
    16:                           restore packages from a paket.references file; may be repeated
    17:     --target-framework <framework>
    18:                           restore only for the specified target framework
    19:     --silent, -s          suppress console output
    20:     --verbose, -v         print detailed information to the console
    21:     --log-file <path>     print output to a file
    22:     --help                display this list of options.
如果添加`--verbose`命令行参数，Paket会在详细模式下运行，并显示详细信息。   
利用`--log-file [path]`命令行参数，可以将日志信息存放到文件中。

利用`paket restore --force`可不可以代替`paket clear-cache`的作用，当上传同名的包后。

### 需要一个有效的**paket.lock**文件
 
如果**paket.lock**文件不存在，**paket restore**操作会失败。这种情况下不会下载任何依赖包。学习如何创建**paket.lock**文件，请查看**paket install**和**paket update**

 

 


 
 


 
 
 


 

 

 


 
    