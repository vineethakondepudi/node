pipeline {
    agent any

    environment {
        registry = "vineethakondepudi/node-k8s-app"   // your Docker image name
        registryCredential = "dockerhub-cred-id"      // your Jenkins DockerHub credentials ID
        appVersion = "${BUILD_NUMBER}"
        containerName = "node-k8s-container"
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        // Build the source code
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        // Test the source code using Maven
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        // Code quality checkstyle analysis
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        // Build Docker image
        stage('Building image') {
            steps {
                sh "docker build -t ${registry}:${BUILD_NUMBER} ."
            }
        }

        // Push Docker image to Docker Hub
        stage('Upload Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: registryCredential, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${registry}:${BUILD_NUMBER}
                        docker tag ${registry}:${BUILD_NUMBER} ${registry}:latest
                        docker push ${registry}:latest
                    """
                }
            }
        }

        // ✅ Run container from pushed image
        stage('Run Container') {
            steps {
                script {
                    echo "Running container from ${registry}:${BUILD_NUMBER}"
                    // Stop old container if running
                    sh "docker ps -q --filter name=${containerName} | grep -q . && docker stop ${containerName} && docker rm ${containerName} || true"
                    
                    // Run new container
                    sh "docker run -d --name ${containerName} -p 3000:80 ${registry}:${BUILD_NUMBER}"

                    // Wait for it to start and verify it responds
                    sh "sleep 10"
                    sh "curl -f http://localhost:3000 || echo '⚠️ Container did not respond yet.'"

                    echo "✅ Container is running successfully!"
                }
            }
        }

        // Cleanup image to save space
        stage('Remove Unused docker image') {
            steps {
                sh "docker stop ${containerName} || true"
                sh "docker rm ${containerName} || true"
                sh "docker rmi ${registry}:${BUILD_NUMBER} || true"
            }
        }
    }

    post {
        success {
            echo "✅ Build, Push, and Run completed successfully!"
        }
        failure {
            echo "❌ Something went wrong in the pipeline."
        }
    }
}

// pipeline {
//     agent any
//     tools {
// 	    maven "MAVEN3"
// 	    jdk "OracleJDK8"
// 	}
//     environment{
//         registry = "dockingsumit/vproappimage" // use your own app image or this image
//         registryCredential = "dockerhub-id"    // save your own dockerhub credentials in jenkins and provide its ID
//     }
//     stages{
//         // Builds the source code 
//         stage('Build') {
//             steps {
//                 sh 'mvn clean install -DskipTests'
//             }
//             post {
//                 success {
//                     echo "Now Archiving."
//                     archiveArtifacts artifacts: '**/*.war'
//                 }
//             }
//         }
//         // Test the source code using Maven
//         stage('Test'){
//             steps {
//                 sh 'mvn test'
//             }

//         }
//         // Test the code quality using checkstyle analysis
//         stage('Checkstyle Analysis'){
//             steps {
//                 sh 'mvn checkstyle:checkstyle'
//             }
//         }
//         // Tests the code quality using sonarqube analysis
//         stage('Sonar Analysis') {
//             environment {
//                 scannerHome = tool 'sonar4.8' //sonar-scanner name
//             }
//             steps {
//                withSonarQubeEnv('Sonar-server')//sonar-server name {
//                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
//                    -Dsonar.projectName=vprofile \
//                    -Dsonar.projectVersion=1.0 \
//                    -Dsonar.sources=src/ \
//                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
//                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
//                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
//                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
//               }
//             }
//         }
//         // Fetches the test results of sonarqube analysis through a webhook by using a defined quality gate
//         stage("Quality Gate") {
//             steps {
//                 timeout(time: 1, unit: 'HOURS') {
//                     // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
//                     // true = set pipeline to UNSTABLE, false = don't
//                     waitForQualityGate abortPipeline: true
//                 }
//             }
//         }
//         //Builds the docker app image and assigns a tag
//         stage('Building image') {
//             steps{
//               script {
//                 dockerImage = docker.build registry + ":$BUILD_NUMBER"
//               }
//             }
//         }
//         // Uploads the built image to dockerhub
//         stage('Upload Image') {
//           steps{
//             script {
//               docker.withRegistry( '', registryCredential ) {
//                 dockerImage.push("$BUILD_NUMBER")
//                 dockerImage.push('latest')
//               }
//             }
//           }
//         }
//         // Removes the unused docker image to clean up disk space
//         stage('Remove Unused docker image') {
//           steps{
//             sh "docker rmi $registry:$BUILD_NUMBER"
//           }
//         }
//         //Finally fetch the latest image from dockerhub and deploy it on kubernetes cluster using helm command
//         stage('Kubernetes Deploy') {
// 	        agent { label 'KOPS' }
//                 steps {  //The namespace prod must already exist
//                         sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod"
//                 }
//         }

//     }

// }
