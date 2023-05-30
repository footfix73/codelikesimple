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
        // Añadir codigo aqui para el primer escenario 
      }
    }
		
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image ...'
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/footfix73/codelikesimple.git']]])
        
        script {
		// Añadir codigo aqui para el segundo escenario 
       } 
      }
    }
	
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        
        script {
		// Añadir codigo aqui para el tercer escenario 
        }
      }
    }

  }
}
