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
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          sh '''
            set -uox pipefail
            mkdir -p reports

            # Terrascan WILL return non-zero for terragoat (by design)
            set +e
            docker run --rm -t \
              --volumes-from jenkins \
              -w "$WORKSPACE" \
              tenable/terrascan scan \
                -i terraform \
                -d terraform \
                -o json \
                --skip-dirs terraform/aws/resources \
              > reports/terrascan_after.json

            rc=$?
            set -e

            echo "Terrascan exit code: $rc"
            echo "$rc" > reports/terrascan_exit_code.txt

            test -s reports/terrascan_after.json
            exit $rc
          '''
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/terrascan*.json,reports/terrascan_exit_code.txt',
                           fingerprint: true,
                           allowEmptyArchive: true
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
