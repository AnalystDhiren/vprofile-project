pipeline {
    agent any
    tools {
        maven "maven"
        jdk "jdk17"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'dhiren'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.80.64'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Build completed successfully.'
                    archiveArtifacts artifacts: '**/*.war'
                }
                failure {
                    echo 'Build failed.'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
            post {
                success {
                    echo 'Tests passed successfully.'
                }
                failure {
                    echo 'Tests failed.'
                }
            }    
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Checkstyle analysis completed successfully.'
                }
                failure {
                    echo 'Checkstyle analysis failed.'
                }
            }
        }
        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=vprofile \
                          -Dsonar.projectVersion=1.0 \
                          -Dsonar.sources=src/ \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.junit.reportPaths=target/surefire-reports/ \
                          -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                          -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    """
                }
            }
        }
    }
}
