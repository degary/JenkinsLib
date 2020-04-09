#!groovy
@Library('sharelib') _
def tools = new org.devops.tools()
def build = new org.devops.build()
def deploy = new org.devops.deploy()
def toemail = new org.devops.toemail()
def gitlab = new org.devops.gitlab()
String buildType = "${env.buildType}"

String buildShell = "${env.buildShell}"

//String buildHosts = "${env.buildHosts}"
String deployHosts = "${env.deployHosts}"
String deployFunc = "${env.deployFunc}"
String branchName = "${env.branchName}"
String srcUrl = "${env.srcUrl}"




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
                    deploy.AnsibleDeploy(deployHosts,deployFunc)
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
