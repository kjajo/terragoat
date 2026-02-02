pipeline {
  agent any

  environment {
    WORKDIR = "/var/jenkins_home/workspace/${JOB_NAME}"
  }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare reports dir') {
      steps {
        sh '''
          rm -rf reports
          mkdir -p reports
        '''
      }
    }

    stage('IaC Scan - Terrascan (Terraform)') {
      steps {
        sh '''
          set -euxo pipefail

          # Baseline scan
          docker run --rm -t \
            --volumes-from jenkins \
            -w "$WORKDIR" \
            -v "$PWD:/iac" -w /iac \
            tenable/terrascan scan -i terraform -d . -o json > reports/terrascan_after.json

          ls -lah reports
          test -s reports/terrascan_after.json
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/terrascan.json', fingerprint: true
        }
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'reports/**/*', fingerprint: true, allowEmptyArchive: true
    }
  }
}
