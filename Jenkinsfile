#!groovy

def student = "agvozdev"
def path="/opt/gradle/gradle-3.4.1/bin/"

node{
    stage ('Preparation (Checking out)'){
    git url: 'https://github.com/alekseygvozdev/mntlab-pipeline', branch: "${student}"
    }

    stage('Building code') {
        sh "${path}gradle build"
    }

    stage ('Testing code'){
      parallel (
        'Unit Tests': { sh "${path}gradle cucumber"},
        'Jacoco Tests': { sh "${path}gradle jacocoTestReport"},
        'Cucumber Tests': { sh "${path}gradle test"}
      )
    }

    stage("Trigger downstream and fetching artifact") {
        build job: "MNTLAB-${student}-child1-build-job", parameters: [string(name: 'BRANCH_NAME', value: "${student}")], wait: true
        step ([$class: 'CopyArtifact',
          projectName: "MNTLAB-${student}-child1-build-job",
          filter: '*.tar.gz']);
    }

    stage('Packaging and publishing results') {
        sh "tar -xf ${student}_dsl_script.tar.gz"
        sh "tar -zcf pipeline-${student}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile -C build/libs gradle-simple.jar"
        archiveArtifacts "pipeline-${student}-${BUILD_NUMBER}.tar.gz"
        }

    stage('Asking for manual approval') {
        input 'Deploy the artifact?'
        }

    stage('Deployment') {
        sh "java -jar build/libs/gradle-simple.jar"
    }

    stage('Sending status') {
        echo "SUCCESS"
    }
}