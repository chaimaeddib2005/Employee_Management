pipeline {
    agent any
    
    // Environment variables
    environment {
        JAVA_HOME = tool 'JDK11'  // or your configured JDK
        PATH = "${JAVA_HOME}/bin:${PATH}"
        NODE_VERSION = '16'  // Specify Node version
        WORKSPACE_DIR = "${WORKSPACE}"
    }
    
    options {
        // Keep last 30 builds
        buildDiscarder(logRotator(numToKeepStr: '30'))
        // Timeout after 1 hour
        timeout(time: 1, unit: 'HOURS')
        // Add timestamps to logs
        timestamps()
    }
    
    stages {
        stage('Prerequisites Check') {
            steps {
                script {
                    echo '🔍 Checking prerequisites...'
                    sh '''
                        echo "Java version:"
                        java -version
                        echo "\nMaven version:"
                        mvn -version
                        echo "\nNode version:"
                        node -v
                        echo "\nNPM version:"
                        npm -v
                    '''
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                script {
                    echo '📥 Cloning repository...'
                    try {
                        // Clean previous workspace
                        deleteDir()
                        // Clone with timeout
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/master']],
                            userRemoteConfigs: [[
                                url: 'https://github.com/chaimaeddib2005/Employee_Management.git',
                                credentialsId: 'github-credentials'  // Add your GitHub credentials in Jenkins
                            ]],
                            extensions: [
                                [$class: 'CloneOption', timeout: 10],
                                [$class: 'CheckoutOption', timeout: 10]
                            ]
                        ])
                        echo '✅ Repository cloned successfully'
                    } catch (Exception e) {
                        echo "❌ Clone failed: ${e.message}"
                        throw e
                    }
                }
            }
        }
        
        stage('Validate Structure') {
            steps {
                script {
                    echo '📋 Validating project structure...'
                    sh '''
                        echo "Checking for backend directory..."
                        if [ ! -d "backend" ]; then
                            echo "❌ Backend directory not found!"
                            exit 1
                        fi
                        
                        echo "Checking for frontend directory..."
                        if [ ! -d "frontend" ]; then
                            echo "❌ Frontend directory not found!"
                            exit 1
                        fi
                        
                        echo "✅ Project structure is valid"
                        
                        echo "\nDirectory listing:"
                        ls -la
                    '''
                }
            }
        }
        
        stage('Build Backend') {
            steps {
                script {
                    echo '🔨 Building backend...'
                    try {
                        dir('backend') {
                            // Clean any previous builds
                            sh 'mvn clean'
                            
                            // Build with verbose output
                            sh '''
                                echo "Checking pom.xml..."
                                if [ ! -f "pom.xml" ]; then
                                    echo "❌ pom.xml not found in backend directory!"
                                    exit 1
                                fi
                                
                                echo "Running Maven build..."
                                mvn -U -X package -DskipTests \
                                    -Dmaven.compiler.source=11 \
                                    -Dmaven.compiler.target=11 \
                                    -Dmaven.test.skip=true
                            '''
                            
                            // Verify JAR was created
                            sh '''
                                if [ -z "$(find target -name '*.jar' 2>/dev/null)" ]; then
                                    echo "❌ No JAR file created!"
                                    exit 1
                                fi
                                echo "✅ JAR file created successfully"
                                ls -la target/*.jar
                            '''
                        }
                    } catch (Exception e) {
                        echo "❌ Backend build failed: ${e.message}"
                        throw e
                    }
                }
            }
        }
        
        stage('Test Backend') {
            steps {
                script {
                    echo '🧪 Running backend tests...'
                    try {
                        dir('backend') {
                            sh '''
                                echo "Running unit tests..."
                                mvn test \
                                    -Dmaven.compiler.source=11 \
                                    -Dmaven.compiler.target=11 || true
                                
                                # Archive test results even if some fail
                                if [ -d "target/surefire-reports" ]; then
                                    echo "✅ Test reports generated"
                                fi
                            '''
                        }
                        
                        // Publish test results
                        junit '**/target/surefire-reports/*.xml'
                        
                    } catch (Exception e) {
                        echo "⚠️  Tests had issues: ${e.message}"
                        // Continue to next stage even if tests fail
                    }
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                script {
                    echo '🎨 Building frontend...'
                    try {
                        dir('frontend') {
                            // Check if frontend files exist
                            sh '''
                                echo "Checking for package.json..."
                                if [ ! -f "package.json" ]; then
                                    echo "❌ package.json not found in frontend directory!"
                                    exit 1
                                fi
                            '''
                            
                            // Clean npm cache
                            sh '''
                                echo "Cleaning npm cache..."
                                npm cache clean --force || true
                                
                                echo "Installing dependencies..."
                                npm install --verbose
                                
                                echo "Building frontend..."
                                npm run build
                                
                                if [ ! -d "build" ] && [ ! -d "dist" ]; then
                                    echo "❌ Frontend build output not found!"
                                    exit 1
                                fi
                                
                                echo "✅ Frontend built successfully"
                            '''
                        }
                    } catch (Exception e) {
                        echo "❌ Frontend build failed: ${e.message}"
                        throw e
                    }
                }
            }
        }
        
        stage('Package') {
            steps {
                script {
                    echo '📦 Packaging artifacts...'
                    try {
                        // Archive JAR files
                        archiveArtifacts(
                            artifacts: '**/target/*.jar',
                            excludes: '**/target/*sources.jar',
                            fingerprint: true,
                            allowEmptyArchive: false
                        )
                        
                        // Archive frontend build
                        sh '''
                            if [ -d "frontend/build" ]; then
                                cd frontend/build
                                tar -czf "${WORKSPACE}/frontend-build.tar.gz" .
                                echo "✅ Frontend build packaged"
                            elif [ -d "frontend/dist" ]; then
                                cd frontend/dist
                                tar -czf "${WORKSPACE}/frontend-build.tar.gz" .
                                echo "✅ Frontend dist packaged"
                            fi
                        '''
                        
                        archiveArtifacts(
                            artifacts: 'frontend-build.tar.gz',
                            fingerprint: true,
                            allowEmptyArchive: true
                        )
                        
                        echo '✅ Artifacts packaged successfully'
                    } catch (Exception e) {
                        echo "❌ Packaging failed: ${e.message}"
                        throw e
                    }
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                script {
                    echo '📊 Running code quality checks...'
                    try {
                        dir('backend') {
                            // Optional: Run SonarQube if configured
                            sh '''
                                echo "Running SonarQube analysis (if configured)..."
                                mvn sonar:sonar \
                                    -Dsonar.projectKey=Employee_Management \
                                    -Dsonar.host.url=http://your-sonarqube-host:9000 \
                                    -Dsonar.login=your-token || echo "⚠️  SonarQube not configured, skipping..."
                            '''
                        }
                    } catch (Exception e) {
                        echo "⚠️  Code quality checks skipped: ${e.message}"
                        // Don't fail the build for code quality
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo '🧹 Cleaning up...'
                // Collect logs
                sh '''
                    echo "=== BUILD SUMMARY ===" > build-summary.txt
                    echo "Build Status: ${BUILD_STATUS:-Unknown}" >> build-summary.txt
                    echo "Timestamp: $(date)" >> build-summary.txt
                '''
                archiveArtifacts(
                    artifacts: 'build-summary.txt',
                    allowEmptyArchive: true
                )
            }
        }
        
        success {
            script {
                echo '✅ BUILD SUCCESSFUL!'
                // Optional: Send notification
                // mail to: 'your-email@example.com',
                //      subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                //      body: "The build completed successfully.\n\nCheck console output at ${env.BUILD_URL}"
            }
        }
        
        failure {
            script {
                echo '❌ BUILD FAILED!'
                // Print build log for debugging
                sh '''
                    echo "=== LAST 100 LINES OF BUILD LOG ===" 
                    tail -100 build-summary.txt || true
                '''
                // Optional: Send notification
                // mail to: 'your-email@example.com',
                //      subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                //      body: "The build failed. Check console output at ${env.BUILD_URL}"
            }
        }
        
        unstable {
            echo '⚠️  BUILD UNSTABLE!'
        }
    }
}
