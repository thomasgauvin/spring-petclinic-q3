def previousSuccessBuildHash = 'null'
def commitsCount = 0
def next = 'null'
def tests_pass = true



pipeline {
  agent any
  stages {
        stage('Copy Archive') {
         steps {
             script {
                step ([$class: 'CopyArtifact',
                    projectName: 'spring-petclinic-q3',
                    filter: "*.txt"]);
            }
          }
        }
    
      stage('Setup Jenkins'){
          steps{
              script{
                if (!fileExists('commitsCount.txt')) {
                  writeFile file: './commitsCount.txt', text: "$commitsCount"
                }

                if (!fileExists('previousSuccessBuildHash.txt')) {
                  writeFile file: './previousSuccessBuildHash.txt', text: "$previousSuccessBuildHash"
                }
              
              }
          }
      }
    
    stage('Previously Success Build Hash Stored?') {
      steps {
        script {
          previousSuccessBuildHash = readFile('previousSuccessBuildHash.txt').trim() 
        }
      }
    }
    stage('8 or more commits?'){
      steps {
        script {
          tmp = readFile('commitsCount.txt').trim()
          echo "$tmp"
          commitsCount= tmp as int
        }
      }
    }
    stage('Build') {
      when {
        expression { previousSuccessBuildHash == 'null' || commitsCount >= 8 }
      }
      steps {
        sh 'mvn clean compile'
        script {
          next = 'test'
        }
      }
    }
    stage('Increase Count') {
      when {
        expression { commitsCount < 8 }
      }
      steps {
        script {
          commitsCount++
          next = 'finish'
        }
        writeFile file: './commitsCount.txt', text: "$commitsCount"
      }
    }
    stage('Test') {
      when {
        expression { next == 'test' }
      }
      steps {
        script {
          try{
            sh 'mvn test'
            script {
              next = 'package'
            }
          }
          catch(e) {
            next = 'finish'
            tests_pass = false 
          }
        }
      }
    }
    stage('Git Bisect') {
      when {
        expression { tests_pass == false && previousSuccessBuildHash != 'null' }
      }
      steps {
        script {
          commitId = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
          sh 'git bisect start "$commitId" "$previousSuccessBuildHash"'
          sh 'git bisect run mvn clean test'
        }
      }
    }
    stage('Package') {
      when {
        expression { next == 'package'  } 
      }
      steps {
        sh 'mvn package'
        script {
          next = 'finish' 
        }
      }
    }
    stage('Finish') {
      when {
        expression { next == 'finish' } 
      }
      steps {
        script {
          if(tests_pass){
            currentBuild.result = "SUCCESS" 
          } else {
            currentBuild.result = "FAILURE"
          } 
        }
      }
    }
  }
    post {
        always {
            archiveArtifacts artifacts: 'previousSuccessBuildHash.txt', onlyIfSuccessful: true
            archiveArtifacts artifacts: 'commitsCount.txt', onlyIfSuccessful: true
        }
    }
}

