#!/usr/bin/groovy
@Library('github.com/hrishin/fabric8-pipeline-library@node-support')
def utils = new io.fabric8.Utils()
nodejsNode{
  def envStage = utils.environmentNamespace('stage')
  def envProd = utils.environmentNamespace('run')
  def newVersion = ''

  def scmVars = checkout scm
  echo "jenkins hash ${scmVars.GIT_COMMIT}"
  echo "jenkins hash ${scmVars}"

  stage('Build Release')
  echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
  if (!fileExists ('Dockerfile')) {
    writeFile file: 'Dockerfile', text: 'FROM node:5.3-onbuild'
  }

  newVersion = performCanaryRelease {}

  echo "new version : ${newVersion}"
  echo "cluster image : ${clusterImageName}"

  def rc = getDeploymentResources {
    port = 8080
    label = 'node'
    icon = 'https://cdn.rawgit.com/fabric8io/fabric8/dc05040/website/src/images/logos/nodejs.svg'
    version = newVersion
    imageName = clusterImageName
  }

  stage('Rollout to Stage')
  kubernetesApply(file: rc, environment: envStage)

  stage('Approve')
  approve{
    room = null
    version = canaryVersion
    console = fabric8Console
    environment = envStage
  }

  stage('Rollout to Run')
  kubernetesApply(file: rc, environment: envProd)

}
