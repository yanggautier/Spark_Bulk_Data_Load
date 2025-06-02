pipeline {
    agent any

    environment {
        PATH = "$HOME/.local/bin:$PATH"
        // Variables d'environnement sécurisées (à définir dans Jenkins)
        DEPLOY_USER = credentials('deploy-username')
        DEPLOY_KEY = credentials('deploy-ssh-key')
        QA_SERVER = credentials('qa-server-ip')
        PROD_SERVER = credentials('prod-server-ip')
    }

    stages {
        stage('Setup Poetry') {
            steps {
                script {
                    def poetryInstalled = sh(
                        script: 'command -v poetry >/dev/null 2>&1',
                        returnStatus: true
                    ) == 0

                    if (!poetryInstalled) {
                        echo "Installation de Poetry..."
                        sh '''
                            python3 -m pip install --user --upgrade pip
                            python3 -m pip install --user pipx
                            python3 -m pipx ensurepath
                            python3 -m pipx install poetry
                        '''
                    }

                    sh 'poetry --version'
                    sh 'poetry config virtualenvs.in-project true'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'poetry install --no-dev'
            }
        }

        stage('Test') {
            steps {
                sh 'poetry run pytest --junitxml=test-results.xml'
            }
            post {
                always {
                    script {
                        if (fileExists('test-results.xml')) {
                            junit 'test-results.xml'
                        }
                    }
                }
            }
        }

        stage('Build Package') {
            when {
                anyOf {
                    branch "main"
                    branch "release/*"
                }
            }
            steps {
                sh '''
                    poetry build
                    zip -r sbdl.zip lib dist/
                '''
            }
        }

        stage('Release to QA') {
            when {
                branch 'release/*'
            }
            steps {
                script {
                    // Utilisation des credentials sécurisés
                    sh """
                        scp -i ${DEPLOY_KEY} \\
                            -o 'StrictHostKeyChecking no' \\
                            -o 'UserKnownHostsFile=/dev/null' \\
                            -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf \\
                            ${DEPLOY_USER}@${QA_SERVER}:/home/${DEPLOY_USER}/sbdl-qa
                    """
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                // Ajouter une confirmation manuelle pour la production
                input message: 'Déployer en production?', ok: 'Déployer',
                      submitterParameter: 'DEPLOYER'
                script {
                    echo "Déploiement approuvé par: ${DEPLOYER}"
                    sh """
                        scp -i ${DEPLOY_KEY} \\
                            -o 'StrictHostKeyChecking no' \\
                            -o 'UserKnownHostsFile=/dev/null' \\
                            -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf \\
                            ${DEPLOY_USER}@${PROD_SERVER}:/home/${DEPLOY_USER}/sbdl-prod
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline terminé avec succès!"
        }
        failure {
            echo "Échec du pipeline - vérifiez les logs"
        }
    }
}