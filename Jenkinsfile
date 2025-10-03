pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/0xhrfugu3/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test || true'  // Allows pipeline to continue despite test failures
      }
      post {
        always {
          emailext (
            subject: "Jenkins Build - Run Tests Stage: ${currentBuild.currentResult}",
            body: """The Run Tests stage completed with status: ${currentBuild.currentResult}.
                     Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]
                     Check the attached log for details.""",
            to: "fderickmarto.tech@gmail.com"
	    attachLog: true
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Ensure coverage report exists
        sh 'npm run coverage || true'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh 'npm audit || true'  // This will show known CVEs in the output
      }
      post {
        always {
          emailext (
            subject: "Jenkins Build - Security Scan Stage: ${currentBuild.currentResult}",
            body: """The NPM Audit stage completed with status: ${currentBuild.currentResult}.
                     Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]
                     Check the attached log for details.""",
            to: "fderickmarto.tech@gmail.com",
            attachLog: true
          )
        }
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            set -eu

            # Pick a known scanner version
            SCANNER_VERSION="5.0.1.3006"
            BASE_URL="https://binaries.sonarsource.com/Distribution/sonar-scanner-cli"
            ZIP="sonar-scanner-cli-${SCANNER_VERSION}-linux.zip"

            echo "Downloading SonarScanner CLI ${SCANNER_VERSION}..."
            curl -sSfL -o "${ZIP}" "${BASE_URL}/${ZIP}"

            echo "Extracting SonarScanner..."
            unzip -q -o "${ZIP}"

            # Find extracted directory dynamically
            SCANNER_DIR="$(find . -maxdepth 1 -type d -name "sonar-scanner-*${SCANNER_VERSION}*" | head -n1)"

            echo "Running SonarScanner..."
            "$SCANNER_DIR/bin/sonar-scanner"
          '''
        }
      }
    }
  }
}
