pipeline {
    agent {
        label 'Slave1'
    }

    tools {
        maven 'maven-3.8.7'
    }

    stages {
        stage('SCM-Checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'git@github.com:manju65char/star-agile-insurance-project.git'
            }
            post {
                failure {
                    sh "echo 'Send mail on failure'"
                    mail to:"dummyid@gmail.com", from: 'dummyid@gmail.com', subject:"FAILURE: ${currentBuild.fullDisplayName}", body: "we failed."
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn compile'
                sh 'mvn test'
                sh 'mvn package'

                stage('Deploy to Ansible Controller ') {
                    steps {
                        script {
                            sshPublisher(publishers: [sshPublisherDesc(configName: 'Ansible-controller', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook  ansible-playbook.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/devopsadmin', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'ansible-playbook.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                        }
                    }
                }
            }
        }
    }
}
