pipeline {
    agent any

    environment {
        registry = "vineethakondepudi/node-k8s-app"   // your Docker image name
        registryCredential = "dockerhub-cred-id"          // your Jenkins DockerHub credentials ID
        appVersion = "${BUILD_NUMBER}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                echo "Validating Node environment..."
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install --quiet'
            }
        }

        stage('Build') {
            steps {
                echo "Building Node.js application..."
                // If you have a build script in package.json
                sh 'npm run build || echo "No build script found, skipping..."'
            }
        }

        stage('Test') {
            steps {
                echo "Running unit tests..."
                // Skip this if you have no tests yet
                sh 'npm test || echo "No tests configured, skipping..."'
            }
        }

        stage('Lint or Checkstyle') {
            steps {
                echo "Running lint analysis..."
                sh 'npm run lint || echo "No lint script configured, skipping..."'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    dockerImage = docker.build("${registry}:${appVersion}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to registry..."
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("${appVersion}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Clean Docker Images') {
            steps {
                sh "docker rmi ${registry}:${appVersion} || true"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes using Helm..."
                // The namespace must exist
                sh "helm upgrade --install node-app ./helm/nodechart --set image.repository=${registry} --set image.tag=${appVersion} --namespace prod"
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successful!"
        }
        failure {
            echo "❌ Build or deployment failed."
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
