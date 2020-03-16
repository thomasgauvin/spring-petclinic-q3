def previousSuccessBuildHash = 'null'
def commitsCount = 0
def next = 'null'
def tests_pass = true



pipeline {
  agent any
  stages {
      stage('Prepare jenkins'){
          steps{
              script{
                  sh 'echo "${commitsCount}"'
                  sh 'echo "${previousSuccessBuildHash}"'

                  sh 'echo "${commitsCount}" > commitsCount.txt'
                  sh 'echo "${previousSuccessBuildHash}" > previousSuccessBuildHash.txt'
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
          tmp = readFile('commitsCount.txt')
          sh 'echo "${commitsCount.txt}"'
          commitsCount = tmp as int
          sh 'echo ${commitsCount}'
        }
      }
    }
    stage('Build') {
      when {
        expression { previousSuccessBuildHash != 'null' || commitsCount >= 8 }
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
        sh 'echo "${commitsCount}" > commitsCount.txt'
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
          sh 'git bisect start ${commitId} ${previousSuccessBuildHash}'
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
}
