#!groovy
def manifest
def environmentcfg

pipeline {
    agent {
        node (){
            label 'buildmachine-java'
        }
    }
    stages {
			stage('Get Production Init') {
				when {
					branch 'task10-master'
            	}
				steps {
					script {
						manifest = readJSON file: 'manifest.json'
						environmentcfg = readJSON file: 'Environment/prod.json'
					}
					echo "Estamos en el branch: ${branch_name}"
				}		
			}
 			stage('Get Stating Init') {
				when {
					not {
							branch 'task10-master'
						}
            	}
				steps {
					script {
						manifest = readJSON file: 'manifest.json'
						environmentcfg = readJSON file: 'Environment/stage.json'
					}
					echo "Estamos en el branch: ${branch_name}"
				}		
			}
			stage('SonarQube Analysis') {
				steps {
					withCredentials([string(credentialsId: 'credUserName', variable: 'NUSER'),string(credentialsId: 'credPassName', variable: 'NPASS')]) {
						sh './gradlew test -PNEXUS_USERNAME=${NUSER} -PNEXUS_PASSWORD=${NPASS}'
					}

					sh "echo TBD"
					sh "mvn -version"
				}
			}
			stage('Build Snapshot & Publish Nexus') {
				when {
					not {
							branch 'task10-master'
						}
            	}
				environment { 
                    VERSIONAPI= sh (returnStdout: true, script: 'curl http://jenkins-api.azurewebsites.net/api/values/newsnapshot/journals').trim()
                }
				steps {
					sh 'echo Generamos un numero nuevo de snapshot, solo se actualiza el valor 1.1.X' 
					sh 'echo Version ${VERSIONAPI}' 
                	sh 'ansible-playbook snapshot.yml --extra-vars "version=${VERSIONAPI} port=${environmentcfg.app.port}"'
				}
			}
			stage('Build Release & Publish Nexus') {
				when {
					branch 'task10-master'
            	}
				environment { 
                    VERSIONAPI= sh (returnStdout: true, script: 'curl http://jenkins-api.azurewebsites.net/api/values/newrelease/journals').trim()
                }
				steps {
					sh 'echo Generamos un numero nuevo de version ya que se trata de un release' 
					sh 'echo Version ${VERSIONAPI}' 
					sh 'ansible-playbook release.yml --extra-vars "version=${VERSIONAPI} port=${environmentcfg.app.port}"'
				}
			}
			stage('Test') {
				steps {
					sh "echo TBD"
				}
			} 
			 
			stage('Deploy') {
				steps {
					sh "echo TBD"
				}
			} 	 
    }
}