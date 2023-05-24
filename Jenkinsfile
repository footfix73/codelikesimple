pipeline {
  agent {
		label 'maven'
	}
  parameters {
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/footfix73/codelikesimple.git', description: 'URL del repositorio Git')
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Rama del repositorio Git')
  }

  stages {
		stage('Build') {
			steps {
				echo 'Building..'

        // Definir el campo spec.source.git con la URL del repositorio git
        spec.source.git = params.GIT_REPO_URL
				sh 'mvn clean package'
			}
		}
		
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image ...'

        // Definir el campo spec.source.git con la URL del repositorio git
        spec.source.git = params.GIT_REPO_URL

				script {
					openshift.withCluster() { 
						openshift.withProject("vicentegarcia-dev") {

							def buildConfigExists = openshift.selector("bc", "codelikesimple").exists() 
							
							if(!buildConfigExists){ 
								openshift.newBuild("--name=codelikesimple", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
							} 
							openshift.selector("bc", "codelikesimple").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") 
            } 
          }
				} 
			}
		}
	
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        
        // Definir el campo spec.source.git con la URL del repositorio git
        spec.source.git = params.GIT_REPO_URL

        script {
          openshift.withCluster() { 
            openshift.withProject("vicentegarcia-dev") { 
              def deployment = openshift.selector("dc", "codelikesimple") 
    
              if(!deployment.exists()){ 
                openshift.newApp('codelikesimple', "--as-deployment-config").narrow('svc').expose() 
              } 
    
              timeout(5) { 
                openshift.selector("dc", "codelikesimple").related('pods').untilEach(1) { 
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