pipeline {
    environment {
        // This registry is important for removing the image after the tests
        registry = "slord0001/jenkins-docker-example"
    }
    agent any
    stages {
        stage("Test") {
            steps {
                script {
                    // Building the Docker image
                    dockerImage = docker.build("$registry")

                    try {
                        dockerImage.inside() {
                            // Extracting the PROJECTDIR environment variable from inside the container
                            def PROJECTDIR = sh(script: 'echo \$PROJECTDIR', returnStdout: true).trim()

                            // Copying the project into our workspace
                            sh "cp -r '$PROJECTDIR' '$WORKSPACE'"

                            // Running the tests inside the new directory
                            dir("$WORKSPACE$PROJECTDIR") {
                                sh "npm test"
                            }
                        }

                    } finally {
                        // push image
                        echo "Trying to Push Docker Build to DockerHub"
                        docker.withRegistry('https://registry.hub.docker.com', 'b5a58bfa-3753-4818-a8cc-e4be44d06a7a') {
                            dockerImage.push("${env.BUILD_NUMBER}")
                            dockerImage.push("latest")
                        } 
                        
                        // Removing the docker image
                        IMAGE_ID="""$(docker images --filter='reference=slord0001/jenkins-docker-example' --quiet)"""
                        echo "Removing Docker Images: ${IMAGE_ID}"
                        sh "docker rmi ${IMAGE_ID} -f"
                    }
                }
            }
        }
    }
}
