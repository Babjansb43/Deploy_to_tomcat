def server = []

pipeline {
    agent any

    environment {
        TOMCAT_USER = credentials('tomcat_credentials')
        TOMCAT_PORT = "8080" // Default Tomcat port
    }

    tools {
        maven 'Maven' 
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Babjansb43/Deploy_to_tomcat.git'
            }
        }
        stage('Capture Version') {
             steps {
                 script {
                     def pom = readMavenPom file: 'pom.xml'
                     env.version = pom.version
                     env.WAR_FILE = "target/sparkjava-hello-world-${env.version}.war"
                 }
             }
        }
        stage('Build') {
            steps {
                script {
                sh 'mvn clean package'
                }
            }
        }
        stage ('Archiving') {
            steps {
            archiveArtifacts artifacts: 'target/*.war', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Deploy WAR file to Tomcat server
                    server = readFile('servers').trim().split('\n')
                    for (host in server) {
                        withEnv(["HOST=${host.trim()}"]) {
                            sh '''
                            curl -u "$TOMCAT_USER_USR:$TOMCAT_USER_PSW" \
                            --upload-file "$WAR_FILE" \
                            "http://$HOST:$TOMCAT_PORT/manager/text/deploy?path=/sparkjava-hello-world&update=true"
                            '''
                        }
                    }
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    def failedServers = []
                    for (host in server) {
                        withEnv(["HOST=${host.trim()}"]) {
                            echo "Verifying application availability on ${host}"
                            def responseCode = sh(
                                script: '''
                                curl -I "http://$HOST:$TOMCAT_PORT/sparkjava-hello-world/hello" \
                                | grep "HTTP" | awk '{print $2}' ''', returnStdout: true ).trim()
                            if (responseCode == "200") {
                                echo "Success! The application on ${host} is up and running."
                            } 
                            else {
                                echo "Deployment verification failed on ${host}. Response code: ${responseCode}"
                                failedServers.add(host)
                            }
                        }
                    }
                    if (!failedServers.isEmpty()) {
                        error(""" Deployment verification failed.The following servers failed:${failedServers.join('\n')}""")
                    }
                    else {
                        echo "Deployment verification completed successfully on all servers."
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Build number ${BUILD_NUMBER} is deployed suucessffully"
        }
        failure {
            echo "Build number ${BUILD_NUMBER} is failed"
        }
    }
}