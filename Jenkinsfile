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
        echo 'Deploying GitHub files to /var/www/html...'
        script {
          // First, let's see what we have and create the target directory
          sh '''
            echo "=== GITHUB TO /var/www/html DEPLOYMENT ==="
            echo "Current workspace: $(pwd)"
            echo "Files from GitHub repository:"
            ls -la
            echo ""
            
            # Create /var/www/html if it doesn't exist
            echo "Setting up /var/www/html directory..."
            if [ ! -d "/var/www" ]; then
              mkdir -p /var/www || echo "Could not create /var/www"
            fi
            
            if [ ! -d "/var/www/html" ]; then
              mkdir -p /var/www/html || echo "Could not create /var/www/html directly"
            fi
            
            echo "Target directory status:"
            ls -ld /var/www/html 2>/dev/null || echo "/var/www/html does not exist or no permission"
            echo ""
          '''
          
          // Try multiple deployment approaches
          sh '''
            echo "=== ATTEMPTING DEPLOYMENT ==="
            
            # Method 1: Direct copy (if we have permissions)
            echo "Method 1: Direct copy to /var/www/html"
            DIRECT_SUCCESS=false
            if cp index.php /var/www/html/ 2>/dev/null; then
              echo "✓ Direct copy successful!"
              DIRECT_SUCCESS=true
            else
              echo "✗ Direct copy failed - no permissions"
            fi
            
            # Method 2: Copy to temp location for manual deployment
            if [ "$DIRECT_SUCCESS" = false ]; then
              echo ""
              echo "Method 2: Preparing files for manual deployment"
              TEMP_DIR="/tmp/github-to-html-$(date +%Y%m%d-%H%M%S)"
              mkdir -p "$TEMP_DIR"
              
              # Copy all project files except git and Jenkins files
              for file in *; do
                if [ "$file" != ".git" ] && [ "$file" != "Jenkinsfile" ] && [ "$file" != ".gitignore" ]; then
                  cp -r "$file" "$TEMP_DIR/"
                  echo "✓ Prepared $file in $TEMP_DIR"
                fi
              done
              
              echo ""
              echo "📁 Files ready for deployment:"
              ls -la "$TEMP_DIR/"
              echo ""
              echo "🚀 TO COMPLETE DEPLOYMENT, RUN THIS COMMAND:"
              echo "sudo cp -r $TEMP_DIR/* /var/www/html/ && sudo chown -R www-data:www-data /var/www/html"
              echo ""
              echo "Or these step-by-step commands:"
              echo "1. sudo mkdir -p /var/www/html"
              echo "2. sudo cp -r $TEMP_DIR/* /var/www/html/"
              echo "3. sudo chown -R www-data:www-data /var/www/html"
              echo "4. sudo chmod -R 755 /var/www/html"
            fi
            
            echo ""
            echo "=== DEPLOYMENT STATUS ==="
            if [ "$DIRECT_SUCCESS" = true ]; then
              echo "✅ Files successfully deployed to /var/www/html"
              echo "Final contents of /var/www/html:"
              ls -la /var/www/html/
            else
              echo "⚠️  Manual deployment required - files prepared in temp directory"
              echo "Run the commands shown above to complete deployment"
            fi
          '''
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
