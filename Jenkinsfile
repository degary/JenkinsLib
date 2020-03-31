#!groovy
@Library('sharelib') _     
def tools = new org.devops.tools()
def build = new org.devops.build()
String buildType = ${env.buildType}
String buildShell = ${env.buildShell}
String deployHosts = ${env.deployHosts}
pipeline {
    agent { node {  label "build01" }}
    stages {
        //下载代码
        stage("build"){ 
            steps{  
                timeout(time:5, unit:"MINUTES"){   
                    script{ 
                        tools.PrintMes("build",'green')
                        build.Build(buildType, buildShell)
                        deploy.AnsibleDeploy("${deployHosts}", "-m ping")
                    }
                }
            }
        }
    }
}
