pipeline {
  agent {
		label 'maven'
	}

  parameters {
    string(name: 'GIT_REPO', description: 'https://github.com/footfix73/codelikesimple.git')
  }

  stages {
		stage('Build') {
			steps {
				echo 'Building..'
				sh 'mvn clean package'
			}
		}
		
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image ...'
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/footfix73/codelikesimple.git']]])

				script {
					openshift.withCluster() { 
						openshift.withProject("vicentegarcia-dev") {

							def buildConfigExists = openshift.selector("bc", "example").exists() 
							
							if(!buildConfigExists){ 
								openshift.newBuild("--name=example", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
							} 
							openshift.selector("bc", "example").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") 
            } 
          }
				} 
			}
		}
	
    stage('Deploy') {
      steps {
        echo 'Deploying....'

        script {
          openshift.withCluster() { 
            openshift.withProject("vicentegarcia-dev") { 
              def deployment = openshift.selector("dc", "example") 
    
              if(!deployment.exists()){ 
                openshift.newApp('example', "--as-deployment-config").narrow('svc').expose() 
              } 
    
              timeout(5) { 
                openshift.selector("dc", "example").related('pods').untilEach(1) { 
                  return (it.object().status.phase == "Running") 
                } 
              }
            }
          }
        }
      }
    }

  }
}