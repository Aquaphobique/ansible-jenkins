pipeline {
    agent any
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
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD'
            }
        }
        stage('Validation') {
            steps {
                sh '''
                    case "$ENVIRONMENT" in
                      dev|test|prod) ;;
                      *) echo "ENVIRONMENT invalide: $ENVIRONMENT"; exit 1 ;;
                    esac
                '''
            }
        }
        stage('Exécution') {
            steps {
                echo "Déploiement env=${params.ENVIRONMENT} version=${params.VERSION} dry_run=${params.DRY_RUN}"
            }
        }
        stage('Post-traitement') {
            steps {
                sh 'mkdir -p artifacts && echo "done" > artifacts/status.txt'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'artifacts/*', allowEmptyArchive: true
        }
        failure {
            echo 'Build en échec — voir les logs ci-dessus'
        }
    }
}
