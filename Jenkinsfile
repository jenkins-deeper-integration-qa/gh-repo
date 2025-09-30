pipeline {
    agent any

    environment {
        // You can set environment variables here
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
    }

      tools {
        maven 'Maven 3'  // Define your Maven installation name from Jenkins Global Tool Configuration
    }

   
    stages {    
       stage('Registering build artifact') {
            steps {
                echo 'Registering the metadata'
                echo 'Another echo to make the pipeline a bit more complex'
                registerBuildArtifactMetadata(
                    name: "test-artifact-cherryl",
                    version: "1.0.0",
                    type: "docker",
                    url: "http://non:1111",
                    digest: "6f637064707039346163663237383938",
                    label: "prod"
                )
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                if ! command -v trivy > /dev/null; then
                  echo "Installing Trivy..."
                  curl -sL https://github.com/aquasecurity/trivy/releases/download/v0.65.0/trivy_0.65.0_Linux-64bit.tar.gz | tar zxvf - -C /tmp
                  mv /tmp/trivy ./trivy
                  chmod +x ./trivy
                fi

                # Run Trivy scan and save SARIF report
                ./trivy fs . --format sarif --output trivy-results.sarif || true
                '''
            }
        }

        stage('Publish Test Results') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }
      
    }
     


    post {
        always {
            echo 'Pipeline completed.'
            archiveArtifacts artifacts: 'trivy-results.sarif', fingerprint: true
        }
        failure {
            echo 'Build or tests failed!'
        }
    }
}
  
