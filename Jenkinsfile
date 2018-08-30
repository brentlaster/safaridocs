#!groovy
pipeline {
   agent { label 'worker_node1' }
   stages{
      stage('Source') {
        steps {
         // always run with a new workspace
         cleanWs()
	 checkout scm
         stash name: 'test-sources', includes: 'build.gradle,src/test/'
        }
      }
      stage('Build') {
        agent {
          dockerfile {
            dir '/home/diyuser2/docker'
          }
        }
        steps {
         // Run the gradle build
         sh 'gradle clean build -x test'         
        }
      }
      stage ('Test') {
        steps {
         // execute required unit tests in parallel	
         parallel (
		worker2: { node ('worker_node2'){
		// always run with a new workspace
			cleanWs()
                        unstash'test-sources'
                      withDockerContainer('gradle:3.4.1') {
			sh 'gradle -D test.single=TestExample1 test'
                      }
		}},
		worker3: { node ('worker_node3'){
		// always run with a new workspace
			cleanWs()
			unstash 'test-sources'             
                      withDockerContainer('gradle:3.4.1') {
			sh 'gradle -D test.single=TestExample3 test'
                      }
		}},
	)
       }    
      } 
   } 
   post {
     always {
       echo "Build stage complete"
     }
     failure {
       echo "Build failed"
       // mail body: 'build failed', subject: 'Build failed!', to: '<your email address>'
     }
     success {
       echo "Build succeeded"
       // mail body: 'build succeeded', subject: 'Build Succeeded', to: '<your email address>'
     }
   }
}


