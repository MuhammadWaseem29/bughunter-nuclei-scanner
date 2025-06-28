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
        sh '''
          git diff --name-only HEAD~1 HEAD | grep "^templates/.*\\.yaml$" > $NEW_TEMPLATES || true
        '''
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
            echo "[*] Running nuclei with $template"
            nuclei -l "$LIVE" -t "$template" -o "scan-$(basename $template .yaml)-$(date +%s).txt"
          done < "$NEW_TEMPLATES"
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: '*.txt', fingerprint: true
    }
  }
}
