pipeline {
    agent none

    stages {
        stage('worker-build') {
            when {
                changeset "**/worker/**"
            }

            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            steps {
                echo 'Compiling worker app...'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        
        stage('worker-test') {
            when {
                changeset "**/worker/**"
            }

            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            steps {
                echo 'Running Unit Tests on worker app...'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        
        stage('worker-package') {
            when {
                branch 'master'
                changeset "**/worker/**"
            }

            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('worker-docker-package') {
            agent any
            
            when {
                branch 'master'
                changeset "**/worker/**"
            }

            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
                        def workerImage = docker.build("sixfold0981/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }


        stage('result-build') {
            when {
                changeset "**/result/**"
            }

            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }

            steps {
                echo 'Compiling result app...'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        
        stage('result-test') {
            when {
                changeset "**/result/**"
            }

            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }

            steps {
                echo 'Running Unit Tests on worker app...'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('result-docker-package') {
            agent any
            
            when {
                branch 'master'
                changeset "**/result/**"
            }

            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
                        def workerImage = docker.build("sixfold0981/result:v${env.BUILD_ID}", "./result")
                        workerImage.push()
                        resultImage.push("${env.BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }

        stage('vote-build') {
            when {
                changeset "**/vote/**"
            }

            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }

            steps {
                echo 'Compiling vote app...'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        
        stage('vote-test') {
            when {
                changeset "**/vote/**"
            }

            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }

            steps {
                echo 'Running Unit Tests on vote app...'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('vote-integration') {
            agent any

            when{ 
                changeset "**/vote/**" 
                branch 'master' 
            }

            steps {
                echo 'Running Integration Tests on vote app' 
                dir('vote') { 
                    sh 'sh integration_test.sh' 
                } 
            } 
        }
        
        stage('vote-docker-package') {
            agent any
            
            when {
                branch 'master'
                changeset "**/vote/**"
            }

            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
                        def voteImage = docker.build("sixfold0981/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                        voteImage.push("latest")
                    }
                }
            }
        }

        stage('Sonarqube') {
            agent any
            when {
                branch 'master'
            }

            environment {
                sonarpath = tool 'SonarScanner'
            }

            steps {
                echo 'Running Sonarqube Analysis..'
                withSonarQubeEnv('sonar-instavote') {
                    sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
                }
            }
        }

        // stage('Quality Gate') {
        //     steps {
        //         waitForQualityGate abortPipeline: true
        //     }
        // }

        stage('deploy-to-dev') {
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
            echo 'Building multibranch pipeline for worker is completed...'
        }
    }
}
