def registry = 'https://deveshdevops.jfrog.io'
def imageName = 'deveshdevops.jfrog.io/valaxy-docker-local/ttrend'
def version   = '2.1.6'


pipeline {
    agent {
        node {
            label 'maven'
        }
    }

environment {
    PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
}    

    stages {
        stage("build"){
            steps {
                sh 'mvn clean deploy'
            }
        }  


         stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifact-cred-new"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "deveshmavenrepo-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->' 
                     echo '<--------------- Jar Publish Ended atlast--------------->' 
            
            }
        }   
    }

    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

    stage (" Docker Publish "){
        steps {
            script {
                echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'artifact-cred-new'){
                app.push()
            }    
                echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }




    stage(" Deploy ") {
      steps {
        script {
          sh './deploy.sh'
        }
      }
    }
 }
}


