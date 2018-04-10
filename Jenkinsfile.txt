#!groovy
/*********************************************************************
***** Description :: This template is used to setup Pipeline *****
***** Date        :: 10/04/2018                                  *****
***** Revision    :: 1.0                                       *****
**********************************************************************/  
import hudson.model.*
import hudson.EnvVars
import groovy.json.*
import java.net.URL


properties ([pipelineTriggers([pollSCM('H/15 * * * *')]), buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5'))])

/****************************************************
***         Other variables
****************************************************/
env.GIT_URL='https://github.com/pragyasanjana/testrepo.git'
/*************************************************
***** Function to checkout the code from Git *****
**************************************************/

 
def  funCodeCheckout()
{
 def WORKSPACE = pwd()
 echo  "\u2600 **********Git Code Checkout Stage Begins*******,GIT_URL=> ${GIT_URL}"
   checkout scm 
   archiveArtifacts allowEmptyArchive: true, artifacts: 'checkout.log', excludes: null
}

def  funCodeBuild()
{
 def WORKSPACE = pwd()
 echo  "\u2600 **********Build started for Visual Studio Project******************"
 dir(PROJECT_PATH){
 bat '"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Professional\\MSBuild\\15.0\\Bin\\MSBuild.exe" Build.proj > build.log || true' 
 bat 'type build.log'
 archiveArtifacts allowEmptyArchive: true, artifacts: 'build.log', excludes: null
 }
}

/********************************************************
*****             Main pipeline stages              *****
*********************************************************/

node('docker-slave') {
    properties([[$class: 'ScannerJobProperty', doNotScan: false], pipelineTriggers([pollSCM('* * * * *')])])
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'amsurgjenkins', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
  {
  wsDir = "workspace/Multibranch.pipeline_${GIT_BRANCH}"
   ws (wsDir) {
   def JAVA_HOME = tool 'JRE1.8-windows'
   env.JAVA_HOME = "$JAVA_HOME"
   env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
   try {
   stage '\u2780 Code Checkout from GIT' 
   funCodeCheckout()
   stage '\u2781 Code Build for Visual Studio Project' 
   funCodeBuild()
   currentBuild.result = 'SUCCESS'
   }
   catch (any) {
        currentBuild.result = 'FAILURE'
        throw any //rethrow exception to prevent the build from proceeding
    }   
  }
  } 
 }