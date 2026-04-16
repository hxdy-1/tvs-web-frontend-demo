pipeline {
  agent any
  parameters {
    string(name: 'SELECTED_BUILD', defaultValue: '', description: 'Build number to release')
    string(name: 'SELECTED_ENV_JOB', defaultValue: 'qa', description: 'Target environment')
  }
  environment {
    PROJECT_NAME    = "tvs-homepage"
    BUILD_PIPELINE_NAME = "tvs-homepage-build"
    AWS_REGION      = "us-east-1"
    BUCKET_NAME     = "tvs-demo-deploy"
    SELECTED_ENV    = "${params.SELECTED_ENV_JOB == 'stage' ? 'qa' : params.SELECTED_ENV_JOB}"
  }
  stages {
    stage('Fetch Artifact') {
      steps {
        script {
          echo "Fetching artifact: ${params.SELECTED_BUILD}"
          copyArtifacts(
            projectName: "${BUILD_PIPELINE_NAME}",
            selector: specific("${params.SELECTED_BUILD}"),
            target: 'artifacts'
          )
          sh "tar -xzf artifacts/${PROJECT_NAME}-${params.SELECTED_BUILD}.tar.gz"
        }
      }
    }
    stage('Deploy to S3') {
        steps {
            script {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh """
                        aws s3 cp publish s3://${BUCKET_NAME}/microfe/${PROJECT_NAME}/ --recursive
                    """
                }
            }
        }
    }
  }
  post {
    success {
      echo "✅ ${PROJECT_NAME} | build ${params.SELECTED_BUILD} | deployed to ${SELECTED_ENV}"
      script {
        currentBuild.displayName = "Release-${params.SELECTED_BUILD}-${currentBuild.number}"
        currentBuild.description = "Build: ${params.SELECTED_BUILD} | Env: ${SELECTED_ENV}"
      }
    }
    always {
      deleteDir()
    }
  }
}