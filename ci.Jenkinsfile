#!groovy
@Library('sharelib') _
//引入tools模块
def tools = new org.devops.tools()
//引入build模块
def build = new org.devops.build()
//引入deploy模块
def deploy = new org.devops.deploy()
//引入tomail模块
def toemail = new org.devops.toemail()
//引入gitlab模块
def gitlab = new org.devops.gitlab()

//定义变量buildType，此变量定义了项目类型 可选参数为  ["mvn":"M2","ant":"ANT","gradle":"GRADLE","cnpm":"NPM"]
String buildType = "${env.buildType}"
//定义jar包名称
String jarName = "${env.jarName}"
//对端接受jar包的目录
String destDir = "${env.destDir}"
//打包参数
String buildShell = "${env.buildShell}"

//对端地址，此变量应设置成与项目名称一样，并在ansible_hosts中添加对应地址
String deployHosts = "${env.deployHosts}"
//ansible 命令参数，暂时没用
String deployFunc = "${env.deployFunc}"
//分支名称。设置成master
String branchName = "${env.branchName}"
//git地址
String srcUrl = "${env.srcUrl}"
//jar包传到对端保存时的名称  jar包名称_构建数
String destName = "${jarName}" + "_${BUILD_NUMBER}"
//脚本放置目录
String scriptDir = "${env.scriptDir}"
//脚本名称   项目名称.sh
String scriptName = "${deployHosts}" + ".sh"



//判断，如果存在runOpts变量，则为gitlabPush请求，否则，为jenkinsPull请求
if (env.runOpts){
    if ("${runOpts}" == "GitlabPush"){
        branchName = branch - "refs/heads/"
        //增加build History中的显示
        currentBuild.description = "Trigger by ${userName} ${branch}"
        //把构建状态推送到gitlab中
        gitlab.ChangeCommitStatus(projectId,commitSha,"running")
        }
    }else{
    if ("${branchName}" == "master"){
        userEmail = "denghui@blhcn.com,huqingnan@blhcn.com"
    }
}

pipeline {
   agent { node {  label "build01" }}

   stages {
      stage('checkout') {
         steps {
             timeout(time:5, unit:"MINUTES"){
                 script{
                    tools.PrintMes("${branchName}", "green")
                    tools.PrintMes("获取代码", "green")
                    checkout([$class: 'GitSCM', branches: [[name: "${branchName}"]], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [], 
                        submoduleCfg: [], 
                        userRemoteConfigs: [[credentialsId: '501bd9e5-1c73-4fe8-9f58-1fd3a9f976c0',
                        url: "${srcUrl}"]]])
                }
            }
         }
      }
      stage('Hello') {
         steps {
             timeout(time:5, unit:"MINUTES"){
                 script{
                    tools.PrintMes("Hello,world","blue")
                 }
             }
         }
      }
      stage('build') {
         steps {
                 script{
                    tools.PrintMes("开始打包", "blue")
                    build.Build(buildType,buildShell)
                 }
         }
      }
      stage('deploy') {
         steps {
             timeout(time:5, unit:"MINUTES"){
                 script{
                    tools.PrintMes("开始发布", "blue")
                    if ("${buildType}" == "mvn"){
	                    sh """
	                    ansible "${deployHosts}" -m copy -a 'src=${WORKSPACE}/target/${jarName} dest=${destDir}/${destName}'
	                    ansible "${deployHosts}" -m shell -a '/bin/bash ${scriptDir}/${scriptName} ${destName}'
	                    """
	                }else if ("${buildType}" == "cnpm"){
	                	sh """
                        tar -zcf "${deployHosts}-${BUILD_NUMBER}".tar.gz dist/
                        ansible "${deployHosts}" -m copy -a 'src=${WORKSPACE}/${deployHosts}-${BUILD_NUMBER}.tar.gz dest=${destDir}'
                        ansible "${deployHosts}" -m shell -a '/bin/bash ${scriptDir}/${scriptName} ${deployHosts}-${BUILD_NUMBER}.tar.gz'
                        """
	                }
                 }
             }
         }
      }
   }
   post{
       always{
           script{
               if (env.runOpts){
                 tools.PrintMes("always", "green")  
               }else{
                tools.PrintMes("else:always", "green")
               }   
           }
       }
       success{
           script{
               if (env.runOpts){
                   tools.PrintMes("success", "green")
                   gitlab.ChangeCommitStatus(projectId,commitSha,"success")  
                   tools.PrintMes("开始发邮件", "green")
                   toemail.Email("构建成功", userEmail)
               }else{
                   tools.PrintMes("发送邮件到: ${userEmail}", "green")
                   toemail.Email("构建成功", userEmail)
               }      
           }
       }
       failure{
           script{
               if (env.runOpts){
                   tools.PrintMes("failure", "red")
                   gitlab.ChangeCommitStatus(projectId,commitSha,"failed")
                   toemail.Email("构建失败了", userEmail)  
               }else{
                   tools.PrintMes("发送邮件到: ${userEmail}", "green")
                   toemail.Email("构建失败", userEmail)
               }
           }
       }
       aborted{
           script{
               if (env.runOpts){
                   tools.PrintMes("aborted", "green1")
                   gitlab.ChangeCommitStatus(projectId,commitSha,"canceled") 
                   toemail.Email("构建被取消了", userEmail) 
               }else{
                   tools.PrintMes("发送邮件到: ${userEmail}", "green")
                   toemail.Email("构建被取消", userEmail)
               }
               
           }
       }
   }
}
