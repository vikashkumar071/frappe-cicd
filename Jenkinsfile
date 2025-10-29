pipeline {
    agent any
    
    environment {
        GIT_REPO_URL = 'https://github.com/roohmeiy/frappe-cicd.git' 
        GIT_BRANCH = 'main' 
        PROJECT_NAME = ''
    }
    
    parameters {
        choice(
            name: 'FRAPPE_BRANCH',
            choices: ['version-15', 'version-14', 'version-13'],
            description: 'Frappe branch (e.g., version-15, version-14)'
        )
        string(
            name: 'SITE_NAME',
            description: 'Site name',
            trim: true
        )
        string(
            name: 'HTTP_PORT',
            description: 'HTTP port for this site (required, e.g., 8080, 8081, 8082)',
            trim: true
        )
        password(
            name: 'DB_PASSWORD',
            defaultValue: '123',
            description: 'Database password (optional, default: 123)'
        )
        password(
            name: 'ADMIN_PASSWORD',
            defaultValue: 'admin',
            description: 'Admin password (optional, default: admin)'
        )
        string(
            name: 'INSTALL_APPS',
            defaultValue: 'frappe',
            description: 'Apps to install (optional, default: frappe)',
            trim: true
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out from: ${env.GIT_REPO_URL}"
                    git branch: env.GIT_BRANCH, url: env.GIT_REPO_URL
                }
            }
        }
        
        stage('Validate apps.json') {
            steps {
                sh '''
                    if [ ! -f "apps.json" ]; then
                        echo "Error: apps.json not found"
                        exit 1
                    fi
                    
                    if ! jq empty apps.json; then
                        echo "Error: Invalid JSON in apps.json"
                        exit 1
                    fi
                    
                    echo "✓ apps.json is valid"
                '''
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Set tag
                    def imageTag = "build-${BUILD_NUMBER}"
                    echo "✓ Image tag: ${imageTag}"
                    
                    // Generate APPS_JSON_BASE64
                    def appsJsonBase64 = sh(
                        script: 'base64 -w 0 apps.json',
                        returnStdout: true
                    ).trim()
                    
                    // Build and push
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        sh """
                            docker build \
                                --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
                                --build-arg=FRAPPE_BRANCH=${params.FRAPPE_BRANCH} \
                                --build-arg=APPS_JSON_BASE64=${appsJsonBase64} \
                                --tag=roohmeiy/frappe-layered:${imageTag} \
                                --file=Dockerfile \
                                .
                            
                            docker push roohmeiy/frappe-layered:${imageTag}
                        """
                    }
                    
                    // Save for deployment stage
                    env.IMAGE_TAG = imageTag
                }
            }
        }
        
        // stage('Generate Project Name') {
        //     steps {
        //         script {
        //             def projectName = params.SITE_NAME
        //                 .replaceAll('[^a-zA-Z0-9]', '-')
        //                 .toLowerCase()
                    
        //             env.PROJECT_NAME = projectName
        //             echo "Project name: ${projectName}"
        //         }
        //     }
        // }
        
        stage('Check Port Conflicts') {
            steps {
                script {
                    def portCheck = sh(
                        script: """
                            if netstat -tuln | grep -q ":${params.HTTP_PORT} "; then
                                echo "CONFLICT"
                            else
                                echo "AVAILABLE"
                            fi
                        """,
                        returnStdout: true
                    ).trim()
                    
                    if (portCheck == "CONFLICT") {
                        error("⚠️  Port ${params.HTTP_PORT} is already in use")
                    } else {
                        echo "✓ Port ${params.HTTP_PORT} is available"
                    }
                }
            }
        }
        
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    def imageTag = env.IMAGE_TAG
                    def projectName = params.SITE_NAME
                        .replaceAll('[^a-zA-Z0-9]', '-')
                        .toLowerCase()
            
                    echo "Deploying ${projectName} on port ${params.HTTP_PORT}"
                    
                    sh """
                        // docker-compose -p "${projectName}" down -v 2>/dev/null || true
                        export ERPNEXT_VERSION="${imageTag}"
                        export SITE_NAME="${params.SITE_NAME}"
                        export HTTP_PUBLISH_PORT="${params.HTTP_PORT}"
                        export DB_PASSWORD="${params.DB_PASSWORD}"
                        export ADMIN_PASSWORD="${params.ADMIN_PASSWORD}"
                        export INSTALL_APPS="${params.INSTALL_APPS}"
                        
                        docker-compose -p "${projectName}" up -d
                        
                        echo "Waiting for services..."
                        sleep 30
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    def maxAttempts = 30
                    def attempt = 0
                    def success = false
                    
                    while (attempt < maxAttempts && !success) {
                        def statusCode = sh(
                            script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${params.HTTP_PORT}/api/method/ping || echo '000'",
                            returnStdout: true
                        ).trim()
                        
                        if (statusCode == '200') {
                            echo "✓ Site responding on port ${params.HTTP_PORT}"
                            success = true
                        } else {
                            attempt++
                            sleep 10
                        }
                    }
                    
                    if (!success) {
                        sh "docker-compose -p '${env.PROJECT_NAME}' logs --tail=100"
                        error("Deployment verification failed")
                    }
                }
            }
        }
        
        stage('Summary') {
            steps {
                script {
                    echo """
                        ✓ Deployment completed!
                        Site: ${params.SITE_NAME}
                        Image: roohmeiy/frappe-layered:${env.IMAGE_TAG}
                        URL: http://localhost:${params.HTTP_PORT}
                        Username: Administrator
                        Password: ${params.ADMIN_PASSWORD}
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '✓ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
