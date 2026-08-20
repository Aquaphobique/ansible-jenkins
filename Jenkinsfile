pipeline {
    agent none
    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'])
        string(name: 'VERSION', defaultValue: '1.0.0')
        booleanParam(name: 'DRY_RUN', defaultValue: true)
        string(name: 'CHANGE_REFERENCE', defaultValue: '')
    }
    stages {
        stage('Préparation') {
            agent any
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD'
            }
        }
        stage('Validation') {
            agent any
            steps {
                sh '''
                    case "$ENVIRONMENT" in
                      dev|test|prod) ;;
                      *) echo "ENVIRONMENT invalide: $ENVIRONMENT"; exit 1 ;;
                    esac
                '''
            }
        }
        stage('AWS - inventaire') {
            agent { label 'aws-lab' }
            steps {
                sh 'aws sts get-caller-identity'
            }
        }
        stage('Exécution') {
            agent any
            steps {
                echo "Déploiement env=${params.ENVIRONMENT} version=${params.VERSION} dry_run=${params.DRY_RUN}"
            }
        }
        stage('Post-traitement') {
            agent any
            steps {
                sh 'mkdir -p artifacts && echo "done" > artifacts/status.txt'
            }
        }
    }
    post {
        always {
            node('') {
                archiveArtifacts artifacts: 'artifacts/*', allowEmptyArchive: true
            }
        }
        failure {
            echo 'Build en échec — voir les logs ci-dessus'
        }
    }
}
