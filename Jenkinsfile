pipeline {
    agent any

    tools {
        maven 'apache-maven-4.0.0-rc-4'
        jdk 'java-17.0.15'
    }

    environment {
        LOCAL_BUILD_DIR = '/tmp/build-netflix-homepage'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/marleneRaissa/Netflix-Homepage.git']]
                ])
            }
        }

        stage('Initialize'){
            steps{
                echo "PATH = ${M2_HOME}/bin:${PATH}"
                echo "M2_HOME = /opt/apache-maven-4.0.0-rc-4"
            }
        }
        
        stage('Build in Temporary Directory') {
            steps {
                script {
                    sh '''
                        set -e
                        rm -rf $LOCAL_BUILD_DIR
                        mkdir -p $LOCAL_BUILD_DIR
                        cp -r $WORKSPACE/* $LOCAL_BUILD_DIR/
                    '''
                    dir("${env.LOCAL_BUILD_DIR}") {
                        sh 'mvn clean install'
                    }
                    // Copier le fichier WAR dans le workspace pour qu'il soit transféré ensuite
                    sh 'mkdir -p $WORKSPACE/target && cp $LOCAL_BUILD_DIR/target/*.war $WORKSPACE/target/'
                }
            }
        }

        stage('Copy Artifact') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'ansible-server',
                            transfers: [
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
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Upload to Docker Hub') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'ansible-server',
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    excludes: '',
                                    execCommand: '''
                                        set -x
                                        cd /opt/docker
                                        docker image prune -a --force
                                        docker build -t my_tomcat_image:v1 .
                                        docker tag my_tomcat_image:v1 marleneraissa/my_tomcat_image_netflix:v1
                                        docker push marleneraissa/my_tomcat_image_netflix:v1
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
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Run Docker Container') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'id-vault-docker',  // ID de vos credentials dans Jenkins
                        variable: 'VAULT_PASS'
                    ) 
                ]) 
                {
                script { echo "Vault password : ${env.VAULT_PASS.length()} characters" }
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'ansible-server',
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    excludes: '',
                                    execCommand: '''
                                        set -x
                                        cd /opt/playbooks || exit 1
                                        VAULT_PASS_FILE=$(mktemp /tmp/.vault_pass_XXXXXX.txt)
                                    
                                        # Écriture sécurisée du mot de passe
                                        echo -n "${env.VAULT_PASS}" > "\${VAULT_PASS_FILE}"
                                        chmod 600 "\${VAULT_PASS_FILE}"
                                    
                                        # Vérification que le fichier n'est pas vide
                                        if [ ! -s "\${VAULT_PASS_FILE}" ]; then
                                           echo "ERROR: Vault password file is empty!"
                                           exit 1
                                        fi
                                        
                                        ansible-playbook start_container.yaml --vault-password-file /tmp/.vault_pass.txt
                                        rm -f "\${VAULT_PASS_FILE}"
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
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true
                        )
                    ]
                )
            }  
            }
        }
    }
}
