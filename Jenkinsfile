pipeline {
  agent any
  environment {
    docker_username = 'lassib'
  }
  stages {
    stage('clone down') {
      steps {
            stash(name: 'code', excludes: '.git')
      }
    }
    stage('say hello') {
      parallel {
        stage('say hello') {
          steps {
            sh '''echo "hello world"'''
          }
        }
        stage('build app') {
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            stash(name: 'code')
            archiveArtifacts 'app/build/libs/'
          }
          options {
            skipDefaultCheckout(true)
          }
        }
        stage('test app') {
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
          options {
            skipDefaultCheckout(true)
          }
        }
      }
    }
    stage('push docker app') {
        environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub
        sh 'ci/push-docker.sh'
      }
    }
  }
  post {
    cleanup {
        deleteDir() /* clean up our workspace */
    }
  }
}
