pipeline {
    agent any
	parameters {
		string(name: 'prod_ip',
			defaultValue: '10.192.10.20',
			description: 'ip address of host server to deploy container to')
	}
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("ukpractice/train-schedule")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy To Prod') {
            when {
                branch 'master'
            }
		steps {
//                input 'Deploy to Prod?'
                milestone(1)
                withCredentials([sshUserPrivateKey(credentialsId: 'swarm_login', keyFileVariable: 'KEYFILE', passphraseVariable: '', usernameVariable: 'USERNAME')]) {
                    script {
                        sh 'sshpass -v ssh -i $KEYFILE -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull ukpractice/train-schedule:${env.BUILD_NUMBER}\"'
                        try {
                            sh 'sshpass -v ssh -i $KEYFILE -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker service rm train-schedule\"'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh 'sshpass -v ssh -i $KEYFILE -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker service create --name train-schedule --replicas 3 -p 80:3000 ukpractice/train-schedule:${env.BUILD_NUMBER}\"'
                    }
                }
            }
        }
		stage('Smoke Test') {
			steps {
				script {
						sh "sleep 30"
						sh "curl -s -o /dev/null -D - $prod_ip:80"
					}
				}
			}
		
	
    }
}
