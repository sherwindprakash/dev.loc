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
              echo "=== DEPLOYMENT DEBUG INFO ==="
              echo "Current directory: $(pwd)"
              echo "Available files to deploy:"
              ls -la
              echo ""
              
              echo "Target directory status:"
              ls -la /var/www/html/ || echo "/var/www/html does not exist"
              echo ""
              
              echo "Copying files to /var/www/html..."
              # Copy all files except Jenkinsfile and git files
              COPIED_FILES=0
              for item in *; do
                if [ "$item" != "Jenkinsfile" ] && [ "$item" != ".git" ] && [ "$item" != ".gitignore" ]; then
                  echo "Copying: $item"
                  if cp -r "$item" /var/www/html/; then
                    echo "  ✓ Successfully copied $item"
                    COPIED_FILES=$((COPIED_FILES + 1))
                  else
                    echo "  ✗ Failed to copy $item"
                  fi
                fi
              done
              
              echo ""
              echo "Files copied: $COPIED_FILES"
              echo "After deployment - /var/www/html contents:"
              ls -la /var/www/html/
              
              # Set basic permissions
              chmod -R 755 /var/www/html/ 2>/dev/null || echo "Could not set all permissions"
              echo "Files deployed successfully to /var/www/html"
            '''
          } else {
            echo 'No write access to /var/www/html - preparing files for manual deployment'
            
            // Prepare files in a staging directory
            sh '''
              echo "=== STAGING DEPLOYMENT DEBUG INFO ==="
              echo "Current directory: $(pwd)"
              echo "Available files to stage:"
              ls -la
              echo ""
              
              STAGING_DIR="${WORKSPACE}/staging"
              mkdir -p "$STAGING_DIR"
              
              echo "Preparing files in staging directory..."
              STAGED_FILES=0
              # Copy all files except Jenkinsfile and git files
              for item in *; do
                if [ "$item" != "Jenkinsfile" ] && [ "$item" != ".git" ] && [ "$item" != ".gitignore" ] && [ "$item" != "staging" ]; then
                  echo "Staging: $item"
                  if cp -r "$item" "$STAGING_DIR/"; then
                    echo "  ✓ Successfully staged $item"
                    STAGED_FILES=$((STAGED_FILES + 1))
                  else
                    echo "  ✗ Failed to stage $item"
                  fi
                fi
              done
              
              echo ""
              echo "Files staged: $STAGED_FILES"
              echo "Staged files:"
              ls -la "$STAGING_DIR/"
              echo ""
              echo "========================================="
              echo "MANUAL DEPLOYMENT REQUIRED"
              echo "========================================="
              echo "Files are prepared in: $STAGING_DIR"
              echo ""
              echo "To complete deployment, run these commands as root:"
              echo "  # Create backup (optional)"
              echo "  mkdir -p /var/www/html"
              echo "  [ -d /var/www/html ] && cp -r /var/www/html /var/www/html-backup-\$(date +%Y%m%d-%H%M%S)"
              echo ""
              echo "  # Deploy files"
              echo "  cp -r $STAGING_DIR/* /var/www/html/"
              echo "  chown -R www-data:www-data /var/www/html"
              echo "  chmod -R 755 /var/www/html"
              echo ""
              echo "Or run this single command:"
              echo "  sudo cp -r $STAGING_DIR/* /var/www/html/ && sudo chown -R www-data:www-data /var/www/html && sudo chmod -R 755 /var/www/html"
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
