#!groovy
@Library('jenkinslib') _     
def tools = new org.devops.tools()
pipeline {
    agent { node {  label "build01" }}
    stages {
        //下载代码
        stage("GetCode"){ 
            steps{  
                timeout(time:5, unit:"MINUTES"){   
                    script{ 
                        tools.PrintMes("获取代码",'green')
                    }
                }
            }
        }
    }
}
