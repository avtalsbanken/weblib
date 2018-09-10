pipeline {
    agent {
      label "jenkins-maven"
    }
    environment {
    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
          PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        }
        steps {
          container('maven') {
            sh "mvn versions:set -DnewVersion=$PREVIEW_VERSION"
            sh "mvn install"
            sh "jx step nexus release"
          }
        }
      }
      stage('Build Release') {
        when {
          not {
            branch 'PR-*'
          }
        }
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"

            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
            sh "mvn clean versions:set -DnewVersion=\$(cat VERSION)"
            sh "mvn deploy"

            sh "git commit -a -m \"release \$(cat VERSION)\""
            sh "git tag -fa v\$(cat VERSION) -m \"Release version \$(cat VERSION)\""
            sh "git push origin v\$(cat VERSION)"

            sh 'updatebot push-version --kind maven se.avtalsbanken:weblib \$(cat VERSION)'
            sh 'updatebot update'
          }
        }
      }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            input """Pipeline failed. 
We will keep the build pod around to help you diagnose any failures. 

Select Proceed or Abort to terminate the build pod"""
        }
    }
  }
