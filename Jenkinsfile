pipeline {
    agent any

    environment {
        // Backend URL where Jenkins reports findings back to the platform
        SECUREDEPLOYHUB_API = "http://backend:8000/api/v1"
        PROJECT_ID = "${env.PROJECT_ID}" // Injected via Jenkins parameter
        PIPELINE_RUN_ID = "${env.PIPELINE_RUN_ID}" // Injected via Jenkins parameter
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from Git
                checkout scm
            }
        }

        stage('SAST - SonarQube') {
            steps {
                script {
                    echo "Running Static Application Security Testing (SAST) via SonarQube..."
                    // Example shell command: 
                    // sh 'mvn sonar:sonar -Dsonar.projectKey=my_project -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=my_token'
                    
                    // Mock reporting a vulnerability back to SecureDeployHub
                    sh """
                    curl -X POST "${SECUREDEPLOYHUB_API}/vulnerabilities/" \
                         -H "Content-Type: application/json" \
                         -d '{
                                "project_id": "'${PROJECT_ID}'",
                                "pipeline_run_id": "'${PIPELINE_RUN_ID}'",
                                "scanner": "SonarQube",
                                "vulnerability_id": "SQ-1234",
                                "title": "SQL Injection",
                                "description": "Unsanitized input in query.",
                                "severity": "CRITICAL",
                                "file_path": "src/main/java/com/example/UserService.java"
                             }'
                    """
                }
            }
        }

        stage('SCA - OWASP Dependency-Check') {
            steps {
                script {
                    echo "Running Software Composition Analysis (SCA) via OWASP Dependency-Check..."
                    // Example shell command:
                    // sh 'dependency-check.sh --project my_project --scan . --format JSON'
                    
                    // Mock reporting a vulnerability back to SecureDeployHub
                    sh """
                    curl -X POST "${SECUREDEPLOYHUB_API}/vulnerabilities/" \
                         -H "Content-Type: application/json" \
                         -d '{
                                "project_id": "'${PROJECT_ID}'",
                                "pipeline_run_id": "'${PIPELINE_RUN_ID}'",
                                "scanner": "OWASP Dependency-Check",
                                "vulnerability_id": "CVE-2021-44228",
                                "title": "Log4Shell",
                                "description": "JNDI features used in configuration, log messages, and parameters do not protect against attacker controlled LDAP and other JNDI related endpoints.",
                                "severity": "CRITICAL",
                                "cvss_score": 10.0,
                                "package_or_component": "log4j-core",
                                "installed_version": "2.14.1",
                                "fixed_version": "2.17.1"
                             }'
                    """
                }
            }
        }

        stage('Container Scan - Trivy') {
            steps {
                script {
                    echo "Running Container Image Scanning via Trivy..."
                    // Example shell command:
                    // sh 'trivy image --format json -o trivy-report.json my-image:latest'
                    
                    // Mock reporting a vulnerability back to SecureDeployHub
                    sh """
                    curl -X POST "${SECUREDEPLOYHUB_API}/vulnerabilities/" \
                         -H "Content-Type: application/json" \
                         -d '{
                                "project_id": "'${PROJECT_ID}'",
                                "pipeline_run_id": "'${PIPELINE_RUN_ID}'",
                                "scanner": "Trivy",
                                "vulnerability_id": "CVE-2023-3817",
                                "title": "OpenSSL memory leak",
                                "severity": "MEDIUM",
                                "package_or_component": "openssl",
                                "installed_version": "3.0.8"
                             }'
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    echo "Checking Quality Gate status against SecureDeployHub policies..."
                    // This calls the backend to evaluate vulnerabilities and creates an Audit Log!
                    sh """
                    curl -X POST "${SECUREDEPLOYHUB_API}/pipelines/runs/${PIPELINE_RUN_ID}/evaluate" \\
                         -H "Content-Type: application/json"
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline complete. Updating status in SecureDeployHub..."
        }
        success {
            echo "Pipeline Passed!"
        }
        failure {
            echo "Pipeline Failed! Security thresholds breached."
        }
    }
}
