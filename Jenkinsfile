pipeline {
    agent any
    environment {
        WEBSERVER = "Nginx"
    }

    options {
        disableConcurrentBuilds()
    }
   
    stages {
        stage('Prepare deployment')
        {
            
            steps{
                sh 'docker info > /dev/null'
                sh 'test -f web/index.html'
                sh 'docker volume create jenkins_web'
            }
        }
        stage('Drop the Apache HTTPD Docker container'){
            when { environment name: 'WEBSERVER', value: 'Apache' }
            steps {
            echo 'droping the container...'
            sh '''
                if docker container inspect apache1 > /dev/null 2>&1; then
                    docker rm -f apache1
                fi
            '''
            }
        }
        stage('Create the Apache httpd container') {
            when { environment name: 'WEBSERVER', value: 'Apache' }
            steps {
            echo 'Creating the container...'
            sh 'docker create --name apache1 -p 9000:80 --mount type=volume,source=jenkins_web,target=/usr/local/apache2/htdocs httpd:2.4'
            }
        }
        stage('Copy the web application to the container directory') {
            when { environment name: 'WEBSERVER', value: 'Apache' }
            steps {
                echo 'Copying web application...'             
                sh 'docker run --rm --volumes-from apache1 httpd:2.4 find /usr/local/apache2/htdocs -mindepth 1 -delete'
                sh 'docker cp web/. apache1:/usr/local/apache2/htdocs/'
                sh 'docker start apache1'
            }
        }

        stage('Drop the Nginx Docker container'){
            when { environment name: 'WEBSERVER', value: 'Nginx' }
            steps {
            echo 'droping the container...'
            sh '''
                if docker container inspect nginx1 > /dev/null 2>&1; then
                    docker rm -f nginx1
                fi
            '''
            }
        }
        stage('Create the Nginx container') {
            when { environment name: 'WEBSERVER', value: 'Nginx' }
            steps {
            echo 'Creating the container...'
            sh 'docker create --name nginx1 -p 9000:80 --mount type=volume,source=jenkins_web,target=/usr/share/nginx/html nginx:latest'
            }
        }
        stage('Copy the web application to the Nginx container directory') {
            when { environment name: 'WEBSERVER', value: 'Nginx' }
            steps {
                echo 'Copying web application...'             
                sh 'docker run --rm --volumes-from nginx1 nginx:latest find /usr/share/nginx/html -mindepth 1 -delete'
                sh 'docker cp web/. nginx1:/usr/share/nginx/html/'
                sh 'docker start nginx1'
            }
        }

        stage('Checking the app') {
            steps {
                echo 'Testing the web app'
                sh 'curl --fail --silent --show-error --retry 10 --retry-connrefused --retry-delay 2 --max-time 10 --output /dev/null http://host.docker.internal:9000/'
            }
        }
        
    }
    post {
            success {
                echo 'Deployment succeeded!'
                archiveArtifacts allowEmptyArchive: true, artifacts: 'web/*', followSymlinks: false
            }
            failure {
                echo 'Deployment failed!'
            }
        }
}
