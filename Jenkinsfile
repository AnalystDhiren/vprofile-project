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
    }

    stages {
        stage('Build'){
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
        stage('test') {
            steps {
                sh 'mvn test'
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
        stage('checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
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
    }
}
