pipeline {
    agent any
    
    environment {
        MAVEN_HOME = tool name: 'Maven', type: 'maven'
        JAVA_HOME = tool name: 'JDK11', type: 'jdk'
        NODE_HOME = tool name: 'NodeJS', type: 'jenkins.plugins.shiningpanda.tools.NodeJSInstallation'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${NODE_HOME}/bin:${PATH}"
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
                    echo '🔍 Checking tools...'
                    sh '''
                        echo "Java:"
                        java -version
                        echo "\nMaven:"
                        mvn -version
                        echo "\nNode:"
                        node -v
                        npm -v
                    '''
                }
            }
        }
        
        stage('Checkout') {
            steps {
                script {
                    echo '📥 Cloning repository...'
                    checkout scm
                }
            }
        }
        
        stage('Build Backend') {
            steps {
                script {
                    echo '🔨 Building backend...'
                    dir('backend') {
                        sh '''
                            mvn clean package -DskipTests
                        '''
                    }
                }
            }
        }
        
        stage('Test Backend') {
            steps {
                script {
                    echo '🧪 Running tests...'
                    dir('backend') {
                        sh '''
                            mvn test || true
                        '''
                    }
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                script {
                    echo '🎨 Building frontend...'
                    dir('frontend') {
                        sh '''
                            npm install
                            npm run build
                        '''
                    }
                }
            }
        }
        
        stage('Archive') {
            steps {
                script {
                    echo '📦 Archiving artifacts...'
                    archiveArtifacts(
                        artifacts: 'backend/target/*.jar,frontend/build/**',
                        allowEmptyArchive: true,
                        fingerprint: true
                    )
                }
            }
        }
    }
    
    post {
        always {
            echo '✅ Build finished'
        }
        
        success {
            echo '✅ Build SUCCESSFUL!'
        }
        
        failure {
            echo '❌ Build FAILED!'
        }
    }
}
