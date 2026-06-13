pipeline {
    agent any

    environment {
        WAR_FILE    = "target/sparkjava-hello-world-1.0.war"
        TOMCAT_USER = credentials('tomcat-username')
        TOMCAT_PASS = credentials('tomcat-password')
        TOMCAT_HOST = "18.61.174.88" // Replace with your Tomcat server's public IP
        TOMCAT_PORT = "8080" // Default Tomcat port
    }

    tools {
        maven 'Maven' 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Babjansb43/Deploy_to_tomcat.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Deploy WAR file to Tomcat server
                    sh '''
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                    --upload-file $WAR_FILE \
                    "http://$TOMCAT_HOST:$TOMCAT_PORT/manager/text/deploy?path=/sparkjava-hello-world&update=true"
                    '''
                }
            }
        }
    }
}