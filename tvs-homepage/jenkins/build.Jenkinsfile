pipeline {
  agent any
  parameters {
    string(name: 'BRANCH', defaultValue: 'develop', description: 'Branch to build')
  }
  environment {
    PROJECT_NAME = "tvs-homepage"
    PROJECT_FOLDER = "tvs-homepage"
    GIT_COMMIT_HASH = GIT_COMMIT.take(6)
    CURRENT_BUILD_DISPLAY = "1.0.${BUILD_NUMBER}-${params.BRANCH}"
  }
  stages {
    stage('Build') {
      steps {
        dir("${PROJECT_FOLDER}") {
          sh '''
            mkdir -p publish
            echo "<html><body>Build: $CURRENT_BUILD_DISPLAY - $(date)</body></html>" > publish/index.html
            tar -zcf ${PROJECT_NAME}-${CURRENT_BUILD_DISPLAY}.tar.gz publish/
          '''
        }
      }
    }
  }
  post {
    always {
      script {
        currentBuild.displayName = "${CURRENT_BUILD_DISPLAY}"
        currentBuild.description = "Branch: ${params.BRANCH} | Commit: ${GIT_COMMIT_HASH}"
      }
    }
    success {
      dir("${PROJECT_FOLDER}") {
        archiveArtifacts artifacts: "${PROJECT_NAME}-${CURRENT_BUILD_DISPLAY}.tar.gz", onlyIfSuccessful: true
      }
    }
  }
}