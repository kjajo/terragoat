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

          # Optional: prove terraform folder exists on Jenkins side
          ls -lah
          ls -lah terraform

          # Share Jenkins container volumes so /var/jenkins_home exists inside the scan container
          docker run --rm -t \
            --volumes-from jenkins \
            -w "$WORKSPACE" \
            tenable/terrascan scan -i terraform -d terraform -o json > reports/terrascan.json

          test -s reports/terrascan.json
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/terrascan.json', fingerprint: true, allowEmptyArchive: true
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
