pipeline {
  agent any

  environment {
    LIVE = "live.txt"
    TEMPLATE_DIR = "templates"
    NEW_TEMPLATES = "new_templates.txt"
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Detect New Templates') {
      steps {
        script {
          def isFirstBuild = currentBuild.previousBuild == null
          if (isFirstBuild) {
            echo "⚠️ First build — scanning all templates in templates/"
            sh "find $TEMPLATE_DIR -name '*.yaml' > $NEW_TEMPLATES"
          } else {
            echo "🔍 Detecting newly added templates..."
            sh "git diff --name-only HEAD~1 HEAD | grep '^templates/.*\\.yaml\$' > $NEW_TEMPLATES || true"
          }
        }
      }
    }

    stage('Run Nuclei Scan') {
      when {
        expression {
          return fileExists(env.NEW_TEMPLATES) && readFile(env.NEW_TEMPLATES).trim() != ""
        }
      }
      steps {
        sh '''
          while read template; do
            echo "[*] Running nuclei scan for $template"
            nuclei -l "$LIVE" -t "$template" -o "scan-$(basename $template .yaml)-$(date +%s).txt"
          done < "$NEW_TEMPLATES"
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'scan-*.txt', fingerprint: true
    }
  }
}
