pipeline {
    agent any
    environment {
        MARIADB_SSH_KEY = credentials('mariadb-ssh-key')
        GCP_SERVICE_ACCOUNT = credentials('gcp-service-account-storage')
        WEBHOOK = credentials('WEBHOOK_URL_DISCORD')
        BACKUP_DIR = "/home/titan/backup-db"
    }
    stages {
        stage('Backup Database') {
            steps {
                echo "Backup..."
                sh "ssh -o StrictHostKeyChecking=no -i $MARIADB_SSH_KEY titan@34.128.74.64 whoami"
                sh 'ssh -o StrictHostKeyChecking=no -i $MARIADB_SSH_KEY titan@34.128.74.64 "sudo mysqldump db_customer > $BACKUP_DIR/backup-users-${BUILD_NUMBER}.sql"'
            }
        }
        stage('Upload to GCS') {
            steps {
                echo "Upload to GCS"
                sh 'scp -o StrictHostKeyChecking=no -i $MARIADB_SSH_KEY $GCP_SERVICE_ACCOUNT titan@34.128.74.64:~/backup-db'
                sh 'ssh -o StrictHostKeyChecking=no -i $MARIADB_SSH_KEY titan@34.128.74.64 "gcloud auth activate-service-account $(cat $BACKUP_DIR/ancient-alloy-406700-4028cf740bc1.json | jq -r .client_email) --key-file=$BACKUP_DIR/ancient-alloy-406700-4028cf740bc1.json"'
                sh 'ssh -o StrictHostKeyChecking=no -i $MARIADB_SSH_KEY titan@34.128.74.64 "gsutil cp $BACKUP_DIR/backup-users-${BUILD_NUMBER}.sql gs://backup-mariadb"'
            }
        }
        stage('Delete all data') {
            steps {
                sh 'ssh -o StrictHostKeyChecking=no -i $MARIADB_SSH_KEY titan@34.128.74.64 "rm -rf $BACKUP_DIR/*"'
            }
        }
    }
    post {
        success {
            echo "Post Success"
            discordSend description: "Jenkins Pipeline Backup", footer: "Backup Success", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "$WEBHOOK"
        }
        aborted {
            echo "Post Aborted"
             discordSend description: "Jenkins Pipeline Backup", footer: "Backup Aborted", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "$WEBHOOK"
        }
        failure {
            echo "Post Failure"
             discordSend description: "Jenkins Pipeline Backup", footer: "Backup Failure", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "$WEBHOOK"
        }
    }
}