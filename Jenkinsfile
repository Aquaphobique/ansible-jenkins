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
                echo "Déploiement env=${params.ENVIRONMENT} version=${params.VERSION} " +
                     "dry_run=${params.DRY_RUN}"
            }
        }
        stage('Ansible - Lint') {
            agent { label 'aws-lab' }
            steps {
                sh 'mkdir -p reports'
                sh '~/.local/bin/ansible-lint ansible/baseline.yml | tee reports/ansible-lint.txt'
            }
        }
        stage('Ansible - Dry run') {
            agent { label 'aws-lab' }
            steps {
                sh '''~/.local/bin/ansible-playbook \
                    -i ansible/inventory.ini ansible/baseline.yml \
                    -e ansible_ssh_private_key_file=~/.ssh/ansible_key --check --diff \
                    | tee reports/ansible-dry-run.txt'''
                archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
            }
        }
        stage('Approbation') {
            agent none
            when { expression { params.ENVIRONMENT == 'prod' } }
            steps {
                input message: "Confirmer l'application sur prod ?"
            }
        }
        stage('Ansible - Exécution réelle') {
            agent { label 'aws-lab' }
            steps {
                sh '~/.local/bin/ansible-playbook \
                    -i ansible/inventory.ini ansible/baseline.yml \
                    -e ansible_ssh_private_key_file=~/.ssh/ansible_key'
            }
        }
        stage('Job aval') {
            agent any
            steps {
                build job: 'romain-tp4-job-aval', parameters: [
                    string(name: 'ENVIRONMENT', value: params.ENVIRONMENT),
                    string(name: 'CHANGE_REFERENCE', value: params.CHANGE_REFERENCE)
                ]
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
                archiveArtifacts(
                    artifacts: 'artifacts/*, ansible/inventory.ini',
                    allowEmptyArchive: true
                )
            }
        }
        failure {
            echo 'Build en échec — voir les logs ci-dessus'
        }
    }
}
