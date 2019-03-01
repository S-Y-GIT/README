# 特别提示 ##
使用该Shared Groovy Libraries需要以下基础依赖项
特别注意基础依赖项之外，针对每个项目的接入需满足以下前提：
> 1. 提前申请制品库（华泰-杜晓丹）
> 2. JENKINS中配置各应用服务器SSH链接
> 3. JENKINS中配置各制品库SSH链接
> 4. Jenkinsfile的配置详细参见下文《Jenkinsfile使用参数说明》（案例参照var目录中的jenkinsfile对应的各txt文件）
> 5. 使用dailyCiPipeline.groovy需满足源码管理分支结构为： master(对应生产环境稳定版本)+开发分支

# 前提依赖 ##
> ##### jenkins版本要求：Jenkins 2.153以上
> ## <依赖1> jenkins插件
> ### 需已安装第三方插件
> * Git plugin
> * Pipeline
> * Pipeline: Groovy
> * Pipeline: Shared Groovy Libraries    2.12
> * Pipeline Maven Integration Plugin    3.6.2    
> * Publish Over SSH    1.20.1    (需版本升级以支持pipeline)
> * HTSC Data Publisher (华泰自研)
> * JaCoCo
> * junit
> * mail
> * Gitlab Merge Request Builder
> 
> ## <依赖2> 编译服务器
> ### 编译服务器需预安装
> * java 1.8.0_121以上
> * maven 3.6.0以上
> * git 2.20 （client以及执行命令）(介质版本依赖于gitlab服务版本目前2.20)
> * python2.7
> #### python2.7模块依赖：
> 系统默认模块：sys,os,shutil,codecs,stat,re,time,datetime,ConfigParser,getopt,zipfile,socket,zipfile,subprocess
> 需安装第三方模块：chardet, git(GitPython), jenkins(python-jenkins)
> #### maven配置依赖：
> 1.编译服务器需安装maven编译环境
> 2.若多maven环境需预先在Jenkins中配置maven版本
> 3.需预先在setting.xml文件中配置sonar访问信息
> #### JDK依赖
> 1.编译服务器需安装指定JDK
> 2.若多jdk环境需预先在Jenkins中配置指定版本
> #### GIT依赖
> 1.安装git客户端可执行命令
> 2.配置SSH密钥连接gitlab项目
>（配置步骤: 1.在编译服务器中生成gitlab服务指定用户（需使用master权限的账号）的SSH-keys密钥；2.在编译服务器中ssh-add添加密钥到密钥管理器ssh-agent中；3.在gitlab服务中添加SSH-keys公钥；4.验证是否成功，执行命令 ssh -T git@gitlab.htzq.htsc.com.cn*）
> 
> ## <依赖3> JENKINS配置
> 1. 需配置指定JDK
> 2. 需配置指定maven
> 3. 需配置各应用服务器SSH链接(*特别注意*)
> 4. 需配置各制品库SSH链接(*特别注意*)
> 5. 需配置全局Pipeline Shared Groovy Libraries
> 
> ## <依赖4> 制品库名称规范
> ##### 注：需要提前申请制品库（杜晓丹）
> - 一级目录为项目所属组别(或能力中心)名称，pipeline中定义为 GROUP_NAME
> - 二级目录为项目名称，pipeline定义为 ARCHIVE_S_NAME ，可自定义，默认为R_${GROUP_NAME}_S_${项目名称}，例如R_CRMMKTG_S_mktg-access

## 共享模块介绍
> ##### 适用范围：git(源码) + maven(编译方式) + java命令直接启动(启动方式)
> ######主要包含阶段[STAGE]为:
> - [CODE] 代码获取并merger
> - [BUILD] 编译并将jar文件和启动脚本、配置文件等静态文件打包
> - [UT] 单元测试
> - [SONAR] SONAR扫描
> - [DEPLOY] 部署项目到服务器
> - [ARCHIVE] 将部署包以及过程中生成的报告文件上传制品库
> - [MAIL] 解析部署日志并发送邮件

## Groovy文件介绍######
> #### Shared Groovy Libraries文件：
> | 文件名  |  文件用途 |
> | ------------ | ------------ |
> |  dailyCiPipeline.groovy  | DEVOPS每日CI脚本PIPELINE文件  |
> | dailyCiPipeline.txt  | DEVOPS每日CI脚本使用案例 |
> | checkBeforeOnline.groovy | 上线前检查脚本PIPELINE文件 |
> | checkBeforeOnline.txt | 上线前检查脚本使用案例 |
> | codeMergeToMaster.groovy | 代码合并到master脚本PIPELINE文件 |
> | codeMergeToMaster.txt | 代码合并到master脚本使用案例 |
> | loadScm.groovy | 预加载SCM脚本文件 |

## Jenkinsfile使用参数说明######
> #### 以dailyCiPipeline.groovy为例，使用案例参照同级目录中的各txt文件
> 以下示例背景说明：
>> 1. 能力中心`CRMINFRA`的项目`crm-auth`
>> 2. 项目路径为`crm-auth/crm-auth-xxx`
>> 3. 项目路径中的主模块路径为`crm-auth/crm-auth-xxx/crm-auth-server`
>> 4. 源码地址为` git@gitlab.htzq.htsc.com.cn:crm/crm-auth.git `，开发分支名称为`develop`
>> 5. maven编译版本为`crm_apache-maven-3.2.5`，java版本为`crm_jdk1.8.0_65`，maven编译参数为`-Xmx1024m`，MAVEN配置文件路径路径为 `http://gitlab.htzq.htsc.com.cn/crminfra/crm-scm/blob/master/maven/settings.xml` 对应的本地路径
>> 6. 需要打包的静态文件目录(位于主模块目录中)为`bin config lib`
>
> ### jenkinsfile使用方式：
> ```groovy
> //设置JENKINS中配置的全局的Pipeline Shared Groovy Libraries
> library 'htsc-ci-pipeline-library'
> //调用Pipeline共享Libs中的指定模块，下文中的部署参数化配置到{...}中，配置参考 daily-ci-pipeline.txt
> dailyCiPipeline{...}
> ```
> ### {}参数化配置定义如下：
| 参数名 | 参数参考值 | 参数定义 | 参数用途 | 
| ------------ | ------------ | -------- | -------- |
| `NODE_SLAVE` |  'master' | **<必填*>** Jenkins中定义的节点服务器名称，用于指定编译服务器的地址。| 后续所有步骤使用 |
| `GROUP_NAME` |  'CRMINFRA' | **<必填*>** 项目所属组别(或能力中心)名称，对应JIRA团队名称。| 1. **步骤 [STAGE-BUILD]中** 编译后打包时使用，定义包名${GROUP_NAME}_S_${PROJECT_NAME}和获取编译生成的jar文件目录${MAIN_SERV}<br>2. **步骤 [STAGE-ARCHIVE]中** 上传制品库时使用，指定制品库二级目录默认值 |
| `PROJECT_NAME` | 'crm-auth' | **<必填*>** 项目应用名称。| 同上 |
| `PROJECT_CODE_ROOT` | '.' | <选填> 项目源码根目录相对路径,默认'.' | 1. **步骤 [STAGE-BUILD]中** 编译时使用，获取源码路径${PROJECT_CODE_ROOT} |
| `MAIN_SERV` | 'crm-auth-server' | <选填> 项目应用中的主程序模块名称,默认'.' | 1. **步骤 [STAGE-BUILD]中** 编译后打包时使用，获取编译生成的jar文件目录${MAIN_SERV} |
| `GIT_URL` | 'git@gitlab.htzq.htsc.com.cn:crm/crm-auth.git' | **<必填*>** 项目应用的源码管理工具git的服务地址。|1. **步骤 [STAGE-CODE]中** 获取源码时使用，获取源码${GIT_URL} |
| `DEFAULT_BRANCH`| 'develop' | <选填> JENKINS传参的默认分支名称，默认值'develop' | 1. **步骤 [STAGE-CODE]中** 获取源码与构建JOB时使用，获取指定默认分支${DEFAULT_BRANCH} |
| `MERGE_FLAG` | 'Y' | <选填> 代码合并标识，默认Y则合并到master，N值则不进行合并 | 1. **步骤 [STAGE-CODE]中** 获取源码后合并项目源码时使用 |
| `JDK_NAME` | 'crm_jdk1.8.0_65' | <选填> 指定jenkins中配置的jdk编译环境版本别名。（默认为操作系统默认设置项） | 1. **步骤 [STAGE-BUILD][STAGE-UT][STAGE-SONAR]中** 指定编译环境时使用，设置${JDK_NAME} ${MAVEN_NAME} ${MAVEN_OPTS} ${MAVEN_CONF} |
| `MAVEN_NAME` | 'crm_apache-maven-3.2.5' | <选填> 指定Jenkins中配置的maven编译环境版本别名。（默认为操作系统默认设置项） | 同上 |
| `MAVEN_OPTS` | '-Xmx1024m' | <选填>MAVEN_OPTS: MAVEN编译配置-OPTS参数，例如：'-Xmx1024m'。| 同上 |
| `MAVEN_CONF` | '/home/jenkins/workspace/CRMMKTG/.crm-scm/maven/settings.xml' | <选填> MAVEN配置文件路径(settings.xml)，默认http://gitlab.htzq.htsc.com.cn/crminfra/crm-scm/blob/master/maven/settings.xml对应的本地绝对路径 ||
| `STATIC_NAME` | 'bin config lib' | <选填> 静态文件目录，多值以空格‘ ’分隔。（位于主模块目录下级目录中） | 1. **步骤 [STAGE-BUILD]中** 编译后打包时使用，获取静态文件目录${STATIC_NAME} |
| `UT_JACOCO_NAME` | 'jacoco_ut.exec' | <选填> UT-单元测试生成的jacoco文件名称。（默认为jacoco_ut.exec） | 1. **步骤 [STAGE-UT][STAGE-SONAR]中** 指定测试报告时使用。 |
| `UT_EXCLUSION` | '**/*Test*.class' | <选填> 生成jacoco报告过程中的排除匹配项。（默认为**/*Test*.class） | 1. **步骤 [STAGE-UT]中** 生成代码覆盖率报告时使用。 |
| `DEPLOY_HOSTS` | 'crm_168.63.65.96' | **<二选一><必填*>** 部署配置-支持两种方式 <br>  方式1: 调用外部JOB方式实现${DEPLOY_JOB} <br>  方式2: 调用jenkins-pipeline内置方法实现${DEPLOY_HOSTS} <br><br>指定jenkins中配置的应用部署服务器的SSH别名，多值以';'分隔 | 1. **步骤 [STAGE-DEPLOY]中** 部署项目应用到指定服务器时使用 |
| `DEPLOY_DIRPATH` | '/app/crm-auth' | **<配合`DEPLOY_HOSTS`使用>** 部署服务器中的项目应用部署目录 | 同上 |
| `DEPLOY_CMDS` | 'sh bin/start.sh restart sit debug' | **<配合`DEPLOY_HOSTS`使用>** 部署服务器中运行的部署服务启停脚本命令 | 同上 |
| `DEPLOY_JOB` | 'DEV_CRMINFRA_S_crm-auth_DEPLOY' | **<二选一>** 指定jenkins中配置项目部署的JOB名称，设置DEPLOY_JOB参数的话其他[DEPLOY]参数将失效 | 同上 |
| `ARCHIVE_HOST` | '制品库（CRMINFRA）' | **<二选一><必填*>** 上传制品库配置-支持两种方式 <br>  方式1: 调用外部JOB方式实现${ARCHIVE_JOB} <br>  方式2: 调用jenkins-pipeline内置方法实现${ARCHIVE_HOST}  <br><br>指定jenkins中配置的项目制品库的SSH别名 | 1. **步骤在 [STAGE-ARCHIVE]中** 上传制品到指定制品库服务器时使用 |
| `ARCHIVE_JOB` | 'DEV_CRMINFRA_S_crm-auth_ARCHIVE' | **<二选一>** 指定jenkins中配置上传制品库的JOB名称，设置ARCHIVE_JOB参数的话其他[ARCHIVE]参数将失效 | 同上 |
| `EMAIL_TO` | 'K0170049@test.htsc.com.cn;K0170050@test.htsc.com.cn' | <选填> 邮件收件人清单，多值;分隔。若不填则不发送邮件 | 1. **步骤 [STAGE-MAIL]中** 在发送邮件时使用 |
|  |  |  |  |
|  |  |  |  |
