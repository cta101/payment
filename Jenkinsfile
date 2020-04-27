// String credentialsId = 'jenkinsAwsCredentials'

def getSha() {
    return sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
}


try {
    stage('checkout') {
        node {
            cleanWs()
            checkout scm
        }
    }

    // Dependencies
    stage('init') {
        node {
            docker.image('ctael5co/golang-git-alpine').inside {
                sh 'go get -u github.com/FiloSottile/gvt'
            }
        }
    }

    stage("build go") {
        node {
            docker.image('golang:latest').inside {
                sh "go get -u github.com/FiloSottile/gvt"
                sh "gvt restore"
            }
        }
    }

    stage("build dockerfile") {
        docker.withRegistry('https://registry.hub.docker.com/', 'dockerhub-ctael5co') {
            shortSHA = getSha()
            def customImage = docker.build("ctael5co/sh-payment:${shortSHA}","./docker/payment/")
            customImage.push()
            customImage.push('latest')    
        }
    }

  currentBuild.result = 'SUCCESS'
}
catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException flowError) {
  currentBuild.result = 'ABORTED'
}
catch (err) {
  currentBuild.result = 'FAILURE'
  throw err
}
finally {
  if (currentBuild.result == 'SUCCESS') {
    currentBuild.result = 'SUCCESS'
  }
}
