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
          mkdir -p reports

          echo "Workspace content:"
          ls -lah
          echo "Terraform files:"
          find . -maxdepth 3 -type f -name "*.tf" -print || true

          docker run --rm \
            -v "$PWD:/iac" -w /iac \
            tenable/terrascan scan -i terraform -d terraform -o json > reports/terrascan.json

          test -s reports/terrascan.json
        '''
        archiveArtifacts artifacts: 'reports/terrascan.json', fingerprint: true
      }
    }



  }
  post {
    always {
      archiveArtifacts artifacts: 'reports/**/*', fingerprint: true, allowEmptyArchive: true
    }
  }
}
