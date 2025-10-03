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
          // Check if we can write to /var/www/html directly
          def canWriteToHtml = sh(script: 'test -w /var/www/html', returnStatus: true) == 0
          
          if (canWriteToHtml) {
            echo 'Jenkins has write access to /var/www/html - deploying directly'
            
            // Create backup of existing files
            sh '''
              if [ -d "/var/www/html" ] && [ "$(ls -A /var/www/html 2>/dev/null)" ]; then
                echo "Creating backup of existing files..."
                BACKUP_DIR="/tmp/html-backup-$(date +%Y%m%d-%H%M%S)"
                mkdir -p "$BACKUP_DIR"
                cp -r /var/www/html/* "$BACKUP_DIR/" 2>/dev/null || echo "Backup completed with warnings"
                echo "Backup created at: $BACKUP_DIR"
              fi
            '''
            
            // Copy files to target directory
            sh '''
              echo "Copying files to /var/www/html..."
              # Copy all files except Jenkinsfile and git files
              for item in *; do
                if [ "$item" != "Jenkinsfile" ] && [ "$item" != ".git" ] && [ "$item" != ".gitignore" ]; then
                  echo "Copying: $item"
                  cp -r "$item" /var/www/html/
                fi
              done
              
              # Set basic permissions
              chmod -R 755 /var/www/html/ 2>/dev/null || echo "Could not set all permissions"
              echo "Files deployed successfully to /var/www/html"
            '''
          } else {
            echo 'No write access to /var/www/html - preparing files for manual deployment'
            
            // Prepare files in a staging directory
            sh '''
              STAGING_DIR="${WORKSPACE}/staging"
              mkdir -p "$STAGING_DIR"
              
              echo "Preparing files in staging directory..."
              # Copy all files except Jenkinsfile and git files
              for item in *; do
                if [ "$item" != "Jenkinsfile" ] && [ "$item" != ".git" ] && [ "$item" != ".gitignore" ] && [ "$item" != "staging" ]; then
                  echo "Preparing: $item"
                  cp -r "$item" "$STAGING_DIR/"
                fi
              done
              
              echo ""
              echo "========================================="
              echo "MANUAL DEPLOYMENT REQUIRED"
              echo "========================================="
              echo "Files are prepared in: $STAGING_DIR"
              echo ""
              echo "To complete deployment, run these commands as root:"
              echo "  # Create backup (optional)"
              echo "  cp -r /var/www/html /var/www/html-backup-\$(date +%Y%m%d-%H%M%S)"
              echo ""
              echo "  # Deploy files"
              echo "  cp -r $STAGING_DIR/* /var/www/html/"
              echo "  chown -R www-data:www-data /var/www/html"
              echo "  chmod -R 755 /var/www/html"
              echo "========================================="
            '''
          }
        }
      }
    }
    
    stage('Verify Deployment') {
      steps {
        echo 'Verifying deployment...'
        script {
          def canWriteToHtml = sh(script: 'test -w /var/www/html', returnStatus: true) == 0
          
          if (canWriteToHtml) {
            sh '''
              echo "Checking deployed files in /var/www/html:"
              ls -la /var/www/html/
              echo "Direct deployment verification completed."
            '''
          } else {
            sh '''
              echo "Checking prepared files in staging:"
              ls -la "${WORKSPACE}/staging/"
              echo "Files are ready for manual deployment."
              echo "Staging verification completed."
            '''
          }
        }
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
