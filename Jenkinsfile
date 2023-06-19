pipeline {
    agent any

    
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/sambit81/Netflix-Homepage.git']]
                ])
            }
        }
        
        
        
        stage('Copy Artifact') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: 'Ansible-Server', transfers: [
                        sshTransfer(
                            cleanRemote: false,
                            excludes: '',
                            execCommand: '',
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
                    sshPublisherDesc(usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)
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
                    sshPublisherDesc(usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)
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
                    sshPublisherDesc(usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)
                ])
            }
        }
    }
}
