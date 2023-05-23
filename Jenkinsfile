#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        
        // Add steps here
      }
    }

    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {
          openshift.withCluster() { 
            openshift.withProject("vicentegarcia-dev") {

              def buildConfigExists = openshift.selector("bc", "codelikesimple").exists() 
    
              if(!buildConfigExists){ 
                openshift.newBuild("--name=codelikesimple", "--docker-image=registry.redhat.io/openjdk/openjdk-11-rhel7", "--binary") 
              } 
    
              openshift.selector("bc", "codelikesimple").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") } }

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