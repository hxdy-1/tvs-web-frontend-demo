pipeline {
  agent any
  parameters {
    booleanParam(name: 'AUTO_DEPLOY_ENABLED', defaultValue: true, description: 'Master toggle — uncheck to disable all auto deployments')
    string(name: 'GIT_BEFORE_SHA', defaultValue: '', description: 'Injected by webhook')
    string(name: 'GIT_AFTER_SHA',  defaultValue: '', description: 'Injected by webhook')
  }
  stages {

    stage('Toggle Check') {
      steps {
        script {
          if (!params.AUTO_DEPLOY_ENABLED) {
            currentBuild.displayName = "DISABLED"
            currentBuild.description = "Auto deploy toggle is OFF"
            echo "⏸ Auto deploy is disabled. Exiting."
            currentBuild.result = 'ABORTED'
            error("Auto deploy disabled.")
          }
        }
      }
    }

    stage('Detect Changed MFEs') {
      steps {
        script {
          checkout scm

          def before = params.GIT_BEFORE_SHA?.trim()
          def after  = params.GIT_AFTER_SHA?.trim()

          if (!before || !after || before == after) {
            echo "No valid SHA range. Skipping."
            env.CHANGED_MFES = ""
            return
          }

          def changed = sh(
            script: "git diff --name-only ${before} ${after}",
            returnStdout: true
          ).trim()

          echo "Changed files:\n${changed}"

          def mfeMap = [
            'tvs-vita'       : [build: 'tvs-vita-build',        release: 'tvs-vita-release'],
            'tvs-homepage'   : [build: 'tvs-homepage-build',    release: 'tvs-homepage-release'],
            'tvs-amsi-stores': [build: 'tvs-amsi-stores-build', release: 'tvs-amsi-stores-release'],
          ]

          def touched = []
          changed.split('\n').each { file ->
            mfeMap.keySet().each { folder ->
              if (file.startsWith("${folder}/") && !touched.contains(folder)) {
                touched.add(folder)
              }
            }
          }

          env.CHANGED_MFES = touched.join(',')
          echo "MFEs to deploy: ${env.CHANGED_MFES ?: 'none'}"
        }
      }
    }

    stage('Build + Release') {
      steps {
        script {
          if (!env.CHANGED_MFES) {
            echo "No MFE changes detected. Nothing to do."
            return
          }

          def mfeMap = [
            'tvs-vita'       : [build: 'tvs-vita-build',        release: 'tvs-vita-release'],
            'tvs-homepage'   : [build: 'tvs-homepage-build',    release: 'tvs-homepage-release'],
            'tvs-amsi-stores': [build: 'tvs-amsi-stores-build', release: 'tvs-amsi-stores-release'],
          ]

          env.CHANGED_MFES.split(',').each { mfe ->
            def jobs = mfeMap[mfe]
            echo "=== Processing: ${mfe} ==="

            def buildResult = build(
              job: jobs.build,
              parameters: [string(name: 'BRANCH', value: 'develop')],
              wait: true,
              propagate: true
            )

            def buildNumber = buildResult.displayName
            echo "Build complete: ${buildNumber}"

            build(
              job: jobs.release,
              parameters: [
                string(name: 'SELECTED_BUILD',   value: buildNumber),
                string(name: 'SELECTED_ENV_JOB', value: 'qa')
              ],
              wait: true,
              propagate: true
            )

            echo "✅ ${mfe} | ${buildNumber} | deployed to demo-qa1"
          }
        }
      }
    }
  }

  post {
    aborted { echo "⏸ Pipeline was aborted (toggle off or no changes)." }
    failure { echo "❌ Orchestrator failed. Check individual pipeline logs above." }
    success { echo "🎉 All changed MFEs built and deployed successfully." }
  }
}