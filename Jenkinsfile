#! /usr/bin/env groovy

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
        echo 'Create Container Image..'
        
        script {
			openshift.withCluster() { 
  				openshift.withProject("jram-dev") {
    				def buildConfigExists = openshift.selector("bc", "my-pipeline").exists() 
    				if(!buildConfigExists){ 
      					openshift.newBuild("--name=my-pipeline", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
    				} 
    				def build = openshift.selector("bc", "my-pipeline").startBuild("--from-file=target/simple-maven-jenkins-0.0.1-SNAPSHOT.war", "--follow")
					build.logs("-f")
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
  				openshift.withProject("jram-dev") { 
    			def deployment = openshift.selector("dc", "simplemaven") 
    			if(!deployment.exists()){ 
      				openshift.newApp('simplemaven', "--as-deployment-config").narrow('svc').expose() 
    			} 
    
    			timeout(5) { 
      				openshift.selector("dc", "simplemaven").related('pods').untilEach(1) { 
        				return (it.object().status.phase == "Running") 
      				} 
    			} 
  			} 
}        }
      }
    }
  }
}