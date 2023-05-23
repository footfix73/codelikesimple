pipeline {
  agent {
		label 'maven'
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
				script {
					openshift.withCluster() { 
						openshift.withProject("vicentegarcia-dev") {

							def buildConfigExists = openshift.selector("bc", "mi-pipeline").exists() 
							
							if(!buildConfigExists){ 
								openshift.newBuild("--name=mi-pipeline", "--docker-image=registry.redhat.io/openjdk/openjdk-11-rhel7", "--binary") 
							} 
							openshift.selector("bc", "mi-pipeline").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") 
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
              def deployment = openshift.selector("dc", "mi-pipeline") 
    
              if(!deployment.exists()){ 
                openshift.newApp('mi-pipeline', "--as-deployment-config").narrow('svc').expose() 
              } 
    
              timeout(5) { 
                openshift.selector("dc", "mi-pipeline").related('pods').untilEach(1) { 
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