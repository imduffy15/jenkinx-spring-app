pipeline {
  agent {
    label "jenkins-maven"
  }
  environment {
    ORG = 'imduffy15'
    APP_NAME = 'jenkinx-spring-app'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
  }
  stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        container('maven') {
          sh "mvn versions:set -DnewVersion=$PREVIEW_VERSION"
          sh "mvn install"
          sh "skaffold version"
          sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
          sh "jx --log-level='debug' --verbose=true step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
          dir('charts/preview') {
            sh "make preview"
            sh "jx --log-level='debug' --verbose=true  preview --app $APP_NAME --dir ../.."
          }
        }
      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {
        container('maven') {

          // ensure we're not on a detached head
//          sh "git checkout master"
//          sh "git config --global credential.helper store"
//          sh "jx --log-level='debug' --verbose=true  step git credentials"

          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
//          sh "mvn versions:set -DnewVersion=\$(cat VERSION)"
//          sh "jx --log-level='debug' --verbose=true  step tag --version \$(cat VERSION)"
//          sh "mvn clean deploy"
//          sh "skaffold version"
//          sh "export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml"
          sh "export VERSION=`cat VERSION`"
          sh "jx --log-level='debug' --verbose=true  step post build --image localhost/test/app:0.0.1"
        }
      }
    }
    stage('Promote to Environments') {
      when {
        branch 'master'
      }
      steps {
        container('maven') {
          dir('charts/jenkinx-spring-app') {
            sh "jx step changelog --version v\$(cat ../../VERSION)"

            sh "ls -lah"
            sh "cat Chart.yaml"
            sh "cat values.yaml"
            
            // release the helm chart
            sh "jx step helm release"

            sh "cat values.yaml"
            
            // promote through all 'Auto' promotion Environments
            //sh "jx --log-level='debug' --verbose=true promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)"
          }
        }
      }
    }
  }
  post {
        always {
          cleanWs()
        }
  }
}
