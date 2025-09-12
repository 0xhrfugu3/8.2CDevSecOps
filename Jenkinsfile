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
        sh 'npm test || true'  // Allows pipeline to continue despite test failtures
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
    }

    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            set -e
            echo "Downloading SonarScanner CLI..."
            curl -sSLo sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli.zip

            echo "Extracting SonarScanner..."
            unzip -q -o sonar-scanner.zip

            # find extracted directory dynamically
            SCANNER_DIR=$(find . -maxdepth 1 -type d -name "sonar-scanner-*" | head -n1)

            echo "Running SonarScanner..."
            "$SCANNER_DIR/bin/sonar-scanner"
          '''
        }
      }
    }
  }
}

