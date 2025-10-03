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
          // Create backup of existing files
          sh '''
            if [ -d "/var/www/html" ]; then
              echo "Creating backup of existing files..."
              sudo mkdir -p /var/www/html-backup-$(date +%Y%m%d-%H%M%S)
              sudo cp -r /var/www/html/* /var/www/html-backup-$(date +%Y%m%d-%H%M%S)/ 2>/dev/null || true
            fi
          '''
          
          // Ensure target directory exists
          sh 'sudo mkdir -p /var/www/html'
          
          // Copy files to target directory
          sh '''
            echo "Copying files to /var/www/html..."
            sudo cp -r * /var/www/html/
            sudo chown -R www-data:www-data /var/www/html
            sudo chmod -R 755 /var/www/html
          '''
        }
      }
    }
    
    stage('Verify Deployment') {
      steps {
        echo 'Verifying deployment...'
        sh '''
          echo "Checking deployed files:"
          ls -la /var/www/html/
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
