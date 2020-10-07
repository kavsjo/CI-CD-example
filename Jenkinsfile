pipeline {

    agent none

    stages {
        stage('build-worker') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }   
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('test-worker') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }   
            steps {
                echo 'Running Unit Tets on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('package-worker') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                branch 'master'
                changeset "**/worker/**"
            }   
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTest'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint:true 
                }
            }
        }
        stage('docker-package-worker') {
            agent any
            when {
                changeset "**/worker/**"
                branch 'master'
            }   
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("kavsjo/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH}")
                    }
                }
            }
        }
        stage('build-result') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }   
            steps {
                echo 'Compiling result app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('test-result') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }   
            steps {
                echo 'Running Unit Tests on result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('docker-package-result') {
            agent any
            when {
                changeset "**/result/**"
                branch 'master'
            }   
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def resultImage = docker.build("kavsjo/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("${env.BRANCH}")
                    }
                }
            }
        }
        stage('build-vote') {
            agent {
                docker {
                    image 'python:3.8.6-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }   
            steps {
                echo 'Compiling vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('test-vote') {
            agent {
                docker {
                    image 'python:3.8.6-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }   
            steps {
                echo 'Running Unit Tests on vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }        
        stage('docker-package-vote') {
            agent any
            when {
                changeset "**/vote/**"
                branch 'master'
            }   
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("kavsjo/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH}")
                    }
                }
            }
        }
        stage('Sonarqube') {

            agent any
            environment {
                sonarpath = tool 'SonarScanner'
            }
            steps {
                echo 'Running Sonarqube Analysis..'
                whithSonarQubeEnv('sonar') {
                    sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                }
            }
        }
        stage('deploy to dev') {
            agent any
            when {
                branch 'master'
            }
            steps {
                echo 'Deploy instavote app with docker compose'
                sh 'docker-compose up -d'
            }
        }
    }        

    post {
        always {      
            echo 'Building multibranch pipeline for instavote is completed..'
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: "instavote-cd", message: "Build Succeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        } 
    }
}