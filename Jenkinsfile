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

/********************************************************
*****             Main pipeline stages              *****
*********************************************************/

node('docker-slave') {
    properties([[$class: 'ScannerJobProperty', doNotScan: false], pipelineTriggers([pollSCM('* * * * *')])])
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'amsurgjenkins', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
  {
  wsDir = "workspace/Multibranch.pipeline"
   ws (wsDir) {
   def JAVA_HOME = tool 'JRE1.8-windows'
   env.JAVA_HOME = "$JAVA_HOME"
   env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
   try {
   stage '\u2780 Code Checkout from GIT' 
   funCodeCheckout()
   currentBuild.result = 'SUCCESS'
   }
   catch (any) {
        currentBuild.result = 'FAILURE'
        throw any //rethrow exception to prevent the build from proceeding
    }   
  }
  } 
 }