pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/sambit81/Hello-World-Maven.git']]
                ])
            }
        }
        
        stage('Build and Test') {
            steps {
                sh 'mvn clean install package'
            }
        }
        
        stage('Copy Artifact') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: 'Ansible-Server', transfers: [
                        sshTransfer(
                            cleanRemote: false,
                            excludes: '',
                            execCommand: 'docker login --username sambits --password-stdin < /etc/docker-password.txt',
                            execTimeout: 120000,
                            flatten: false,
                            makeEmptyDirs: false,
                            noDefaultExcludes: false,
                            patternSeparator: '[, ]+',
                            remoteDirectory: '//opt//docker',
                            remoteDirectorySDF: false,
                            removePrefix: 'target',
                            sourceFiles: 'target/*.war'
                        )
                    ]),
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: false
                ])
            }
        }

        stage('Upload to Docker Hub') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: 'Ansible-Server', transfers: [
                        sshTransfer(
                            cleanRemote: false,
                            excludes: '',
                            execCommand: '''
                                cd /opt/docker
                                docker build -t sambits/netflix:latest .
                                docker push sambits/netflix:latest
                            ''',
                            execTimeout: 120000,
                            flatten: false,
                            makeEmptyDirs: false,
                            noDefaultExcludes: false,
                            patternSeparator: '[, ]+',
                            remoteDirectory: '',
                            remoteDirectorySDF: false,
                            removePrefix: '',
                            sourceFiles: ''
                        )
                    ]),
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: false
                ])
            }
        }
        
        stage('Run Docker Container') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: 'Ansible-Server', transfers: [
                        sshTransfer(
                            cleanRemote: false,
                            excludes: '',
                            execCommand: '''
                                cd /opt/playbooks/
                                ansible-playbook start_container.yaml
                            ''',
                            execTimeout: 120000,
                            flatten: false,
                            makeEmptyDirs: false,
                            noDefaultExcludes: false,
                            patternSeparator: '[, ]+',
                            remoteDirectory: '',
                            remoteDirectorySDF: false,
                            removePrefix: '',
                            sourceFiles: ''
                        )
                    ]),
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: false
                ])
            }
        }
    }
}
