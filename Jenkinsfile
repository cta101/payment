// String credentialsId = 'jenkinsAwsCredentials'

def dockerImage = "ctael5co/sh-payment"
def shortSHA = ""

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

    // stage("build go") {
    //     node {
    //         // Either this XDG_CACHE_HOME which sets the go .cache folder
    //         // or run with -u root.  without these it can only write to //// the folders mounted with rw,z 
    //         withEnv(["XDG_CACHE_HOME=/tmp"]) {
    //             docker.image('golang:latest').inside() {
    //                 sh "pwd"
    //                 sh "ls -l"
    //                 sh "go get -u github.com/FiloSottile/gvt"
    //                 sh "gvt restore"
    //             }
    //         }
    //     }
    // }

    stage("build docker") {
        node {
            docker.withRegistry('https://registry.hub.docker.com/', 'dockerhub-ctael5co') {
                shortSHA = getSha()
                def customImage = docker.build("$dockerImage" + ":${shortSHA}","-f docker/payment/Dockerfile .")
                customImage.push()
                customImage.push('latest')    
            }   
        }

    }

    stage("test") {
        node {
            withEnv(["XDG_CACHE_HOME=/tmp"]) {
                docker.image("$dockerImage:$shortSHA").inside() {
                    sh "env && pwd && ls -l && ls -l /go/src/github.com/microservices-demo/payment && ls -l /go/src/github.com/microservices-demo/payment/vendor/github.com/"
                    sh "cd /go/src/github.com/microservices-demo/payment"
                    sh "go test -v -covermode=count -coverprofile=/tmp/coverage.out"
                }
            }
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
