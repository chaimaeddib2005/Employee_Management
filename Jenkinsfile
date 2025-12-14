pipeline {
    agent any
    
    environment {
        MAVEN_HOME = tool name: 'Maven', type: 'maven'
        JAVA_HOME = tool name: 'JDK17', type: 'jdk'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${PATH}"
        DOCKERHUB_USERNAME = 'misswolf4'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 2, unit: 'HOURS')
        timestamps()
    }

    stages {
        stage('Verify Tools') {
            steps {
                script {
                    echo 'Checking tools...'
                    sh '''
                        echo "=== Java Version ==="
                        java -version
                        echo -e "\\n=== Maven Version ==="
                        mvn -version
                        echo -e "\\n=== Node Version ==="
                        node -v || echo " Node not found in PATH"
                        echo -e "\\n=== NPM Version ==="
                        npm -v || echo " NPM not found in PATH"
                        echo -e "\\n=== Docker Version ==="
                        docker --version || echo " Docker not found in PATH"
                    '''
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    echo ' Cloning repository...'
                    checkout scm
                    echo ' Repository cloned'
                }
            }
        }

        stage('Validate Structure') {
            steps {
                script {
                    echo ' Validating project structure...'
                    sh '''
                        echo "Current directory: $(pwd)"
                        echo -e "\\n=== Root contents ==="
                        ls -la

                        echo -e "\\n=== Checking backend ==="
                        if [ -d "backend" ]; then
                            echo " Backend directory exists"
                            if [ -f "backend/pom.xml" ]; then
                                echo " backend/pom.xml found"
                            else
                                echo "backend/pom.xml NOT found"
                                exit 1
                            fi
                        else
                            echo " Backend directory NOT found"
                            exit 1
                        fi

                        echo -e "\\n=== Checking frontend ==="
                        if [ -d "frontend" ]; then
                            echo " Frontend directory exists"
                            if [ -f "frontend/package.json" ]; then
                                echo " frontend/package.json found"
                            else
                                echo " frontend/package.json NOT found"
                                exit 1
                            fi
                        else
                            echo " Frontend directory NOT found"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    try {
                        echo '🔨 Building backend...'
                        dir('backend') {
                            sh '''
                                echo "Running Maven clean package..."
                                mvn clean package -DskipTests -U

                                echo -e "\\n=== Checking for JAR ==="
                                if [ -f "target"/*.jar ]; then
                                    echo " JAR file created:"
                                    ls -lh target/*.jar
                                else
                                    echo "No JAR file created"
                                    exit 1
                                fi
                            '''
                        }
                    } catch (Exception e) {
                        echo " Backend build failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Backend build failed")
                    }
                }
            }
        }
        stage('Test Backend') {
                    steps {
                        script {
                            echo ' Running backend tests...'
                            dir('backend') {
                                sh '''
                                    echo "Running Maven tests..."
                                    mvn test || true
                                '''
                            }
                        }
                    }
                }
        stage('SonarQube Analysis Backend') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        dir('backend') {
                            sh """
                                mvn clean verify sonar:sonar \
                                -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            """
                        }
                    }
                }
            }
        }




        stage('Build Frontend') {
            steps {
                script {
                    try {
                        echo ' Building frontend...'
                        dir('frontend') {
                            sh '''
                                echo "=== Installing dependencies ==="
                                npm install

                                echo -e "\\n=== Building frontend ==="
                                npm run build

                                echo -e "\\n=== Checking build output ==="
                                if [ -d "build" ]; then
                                    echo " Frontend build directory created"
                                    ls -la build/ | head -10
                                elif [ -d "dist" ]; then
                                    echo " Frontend dist directory created"
                                    ls -la dist/ | head -10
                                else
                                    echo " Neither build nor dist directory found"
                                    exit 1
                                fi
                            '''
                        }
                    } catch (Exception e) {
                        echo " Frontend build failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Frontend build failed")
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                script {
                    echo ' Archiving artifacts...'
                    try {
                        archiveArtifacts(
                            artifacts: 'backend/target/*.jar,frontend/build/**,frontend/dist/**',
                            excludes: 'backend/target/*sources.jar',
                            allowEmptyArchive: true,
                            fingerprint: true
                        )
                        echo ' Artifacts archived'
                    } catch (Exception e) {
                        echo " Warning: Could not archive some artifacts: ${e.message}"
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo ' Building Docker images...'

                    // Backend image
                    sh 'docker build -t employees-backend:latest ./backend'

                    // Frontend image
                    sh 'docker build -t employees-frontend:latest ./frontend'

                    echo ' Docker images built successfully'
                }
            }
        }

        stage('Publish Docker Images') {
            steps {
                script {
                    echo ' Publishing Docker images to Docker Hub...'

                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Login
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'

                        // Tag images
                        sh '''
                            docker tag employees-backend:latest $DOCKER_USER/employees-backend:latest
                            docker tag employees-frontend:latest $DOCKER_USER/employees-frontend:latest
                        '''

                        // Push images
                        sh '''
                            docker push $DOCKER_USER/employees-backend:latest
                            docker push $DOCKER_USER/employees-frontend:latest
                        '''
                    }

                    echo ' Docker images pushed to Docker Hub successfully'
                }
            }
        }
      stage('Deploy to Kubernetes') {
          steps {
              sh 'kubectl apply -f Kubernetes/.'
          }
      }
   stage('Deploy Monitoring') {
       steps {
           sh '''
               helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
               helm repo update

               helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
                   --namespace monitoring --create-namespace
           '''
       }
   }



    }

    post {
        always {
            echo ' Cleanup...'
        }

        success {
            echo ' BUILD SUCCESSFUL! 🎉'
        }

        failure {
            echo ' BUILD FAILED! ❌'
            script {
                sh '''
                    echo "=== Build Failed Summary ==="
                    echo "Check workspace: /var/lib/jenkins/workspace/Employees_Manager"
                    echo "To debug, SSH to Jenkins server and check:"
                    echo "  - Backend: ls -la backend/target/"
                    echo "  - Frontend: ls -la frontend/build/"
                '''
            }
        }
    }
}
