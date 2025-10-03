pipeline {
  agent any
  
  stages {
    stage('Initialize') {
      steps {
        echo 'Starting deployment pipeline...'
        echo "Deploying from workspace: ${WORKSPACE}"
        echo "Target directory: /var/www/html"
      }
    }
    
    stage('Checkout') {
      steps {
        echo 'Checking out source code...'
        checkout scm
      }
    }
    
    stage('Build') {
      steps {
        echo 'Building application...'
        // Add any build steps here if needed
        sh 'echo "Build completed successfully"'
      }
    }
    
    stage('Deploy') {
      steps {
        echo 'Deploying to /var/www/html...'
        script {
          // Check if we can write to /var/www/html or need to use alternative approach
          sh '''
            echo "Checking deployment target..."
            
            # Try to create a test file to check permissions
            if touch /var/www/html/jenkins-test 2>/dev/null; then
              echo "Jenkins has write access to /var/www/html"
              rm -f /var/www/html/jenkins-test
              DEPLOY_DIR="/var/www/html"
            else
              # If no direct access, deploy to Jenkins workspace and provide instructions
              echo "No direct write access to /var/www/html"
              echo "Deploying to Jenkins workspace for manual deployment"
              DEPLOY_DIR="${WORKSPACE}/deploy"
              mkdir -p "$DEPLOY_DIR"
            fi
            
            echo "DEPLOY_DIR=$DEPLOY_DIR" > deployment.env
          '''
          
          // Read the deployment directory
          def deploymentEnv = readFile('deployment.env').trim()
          def deployDir = deploymentEnv.split('=')[1]
          
          // Create backup if deploying directly
          sh '''
            source deployment.env
            if [ "$DEPLOY_DIR" = "/var/www/html" ] && [ -d "/var/www/html" ]; then
              echo "Creating backup of existing files..."
              BACKUP_DIR="/var/www/html-backup-$(date +%Y%m%d-%H%M%S)"
              mkdir -p "$BACKUP_DIR" 2>/dev/null || echo "Could not create backup directory"
              cp -r /var/www/html/* "$BACKUP_DIR/" 2>/dev/null || echo "Backup completed with some warnings"
            fi
          '''
          
          // Deploy files
          sh '''
            source deployment.env
            echo "Deploying files to: $DEPLOY_DIR"
            
            # Ensure target directory exists
            mkdir -p "$DEPLOY_DIR"
            
            # Copy files (excluding Jenkins-specific files)
            for file in *; do
              if [ "$file" != "deployment.env" ] && [ "$file" != "Jenkinsfile" ]; then
                echo "Copying: $file"
                cp -r "$file" "$DEPLOY_DIR/"
              fi
            done
            
            echo "Files copied successfully to $DEPLOY_DIR"
          '''
          
          // Set permissions if we have access
          sh '''
            source deployment.env
            if [ "$DEPLOY_DIR" = "/var/www/html" ]; then
              echo "Setting basic permissions..."
              chmod -R 755 "$DEPLOY_DIR" 2>/dev/null || echo "Could not set all permissions - may need manual adjustment"
            else
              echo ""
              echo "=== MANUAL DEPLOYMENT REQUIRED ==="
              echo "Files are ready in: $DEPLOY_DIR"
              echo "To complete deployment, run as root or with sudo:"
              echo "  cp -r $DEPLOY_DIR/* /var/www/html/"
              echo "  chown -R www-data:www-data /var/www/html"
              echo "  chmod -R 755 /var/www/html"
              echo "=================================="
            fi
          '''
        }
      }
    }
    
    stage('Verify Deployment') {
      steps {
        echo 'Verifying deployment...'
        sh '''
          source deployment.env
          echo "Checking deployed files in: $DEPLOY_DIR"
          ls -la "$DEPLOY_DIR/"
          
          if [ "$DEPLOY_DIR" = "/var/www/html" ]; then
            echo "Direct deployment completed successfully!"
          else
            echo "Files prepared for manual deployment."
            echo "Location: $DEPLOY_DIR"
          fi
          
          echo "Deployment verification completed."
        '''
      }
    }
  }
  
  post {
    success {
      echo 'Deployment completed successfully!'
    }
    failure {
      echo 'Deployment failed!'
    }
    always {
      echo 'Pipeline execution finished.'
    }
  }
}
