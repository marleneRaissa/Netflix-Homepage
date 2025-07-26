pipeline {
    agent any

    environment {
        LOCAL_BUILD_DIR = '/tmp/build-netflix-homepage'
    }

    tools {
        maven 'apache-maven-4.0.0-rc-4'
	jdk 'java-17.0.15'
    }
	
    stages {  //Declarative checkout is already configure in pipeline

//        stage('Checkout Git') {
//            steps {
//                checkout([
//                    $class: 'GitSCM',
//                    branches: [[name: '*/main']],
//                    extensions: [],
//                    userRemoteConfigs: [[url: 'https://github.com/marleneRaissa/Netflix-Homepage.git']]
//                ])
//            }
//        }

//	stage('Initialize'){ // optional
//            steps{
//                echo "PATH = ${M2_HOME}/bin:${PATH}"
//                echo "M2_HOME = /opt/apache-maven-4.0.0-rc-4"
//            }
//      }
	    
        stage('Build and Test in Temporary Directory') {
            steps {
                script {
                    sh '''
                        set -e
                        rm -rf $LOCAL_BUILD_DIR
                        mkdir -p $LOCAL_BUILD_DIR
                        cp -r $WORKSPACE/* $LOCAL_BUILD_DIR/
                    '''
                    dir("${env.LOCAL_BUILD_DIR}") {
                        sh 'clean install -DskipTests'
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
	    
	stage('Upload to DockerHub - Run Docker Container') {
    	    steps {
		withCredentials([
		            string(
		                credentialsId: 'id-vault-docker',
		                variable: 'VAULT_PASS'
		            )
		]) {
		      script {
		        def remoteCommand = """
		            echo "[INFO] Écriture du mot de passe vault dans un fichier temporaire..."
		            if [ -z "${VAULT_PASS}" ]; then
		                echo "ERROR: VAULT_PASS is empty!"
		            exit 1
		            fi
		            echo "${VAULT_PASS}" > /tmp/.vault_pass.txt
		            chmod 600 /tmp/.vault_pass.txt
		            ansible-playbook /opt/playbooks/start_container.yaml --vault-password-file /tmp/.vault_pass.txt
		            rm -f /tmp/.vault_pass.txt
		        """
		
		                sshPublisher(
		                    publishers: [
		                        sshPublisherDesc(
		                            configName: 'ansible-server',
		                            transfers: [
		                                sshTransfer(
		                                    execCommand: remoteCommand,
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
		                            verbose: true
		                        )
		                    ]
		                )
		            }
		   }
    	     }
	}
    }
}
