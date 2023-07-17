pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven3.9.3"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                // Checkout the repository into the workspace
                checkout scm
            }
        }
        stage('Maven Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage("Docker build") {
            steps {
                sh 'docker version'
                sh "docker build -t skbrnrgr/insuranceapp:${BUILD_NUMBER} ."
                sh 'docker image list'
                sh "docker tag skbrnrgr/insuranceapp:${BUILD_NUMBER} skbrnrgr/insuranceapp:latest"
            }
        }
        stage('Login to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Approve - push Image to Docker Hub') {
            steps {
                //----------------send an approval prompt-------------
                script {
                    env.APPROVED_DEPLOY = input message: 'User input required. Choose "yes" to approve or "Abort" to reject', ok: 'Yes', submitterParameter: 'APPROVER'
                }
                //-----------------end approval prompt------------
            }
        }
        stage('Push to Docker Hub') {
            steps {
                sh "docker push skbrnrgr/insuranceapp:latest"
            }
        }
        stage('Approve - Deployment to Kubernetes Cluster') {
            steps {
                //----------------send an approval prompt-----------
                script {
                    env.APPROVED_DEPLOY_KUBE = input message: 'User input required. Choose "yes" to approve or "Abort" to reject', ok: 'Yes', submitterParameter: 'APPROVER_KUBE'
                }
                //-----------------end approval prompt------------
            }
        }
        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'kumaster', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f k8sdeployment.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'k8sdeployment.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                }
            }
        }
    }
}
