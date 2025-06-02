pipeline {
    agent any

    environment {
        // Définir le PATH pour inclure les binaires locaux de l'utilisateur
        PATH = "$HOME/.local/bin:$PATH"
    }

    stages {
        stage('Setup Poetry') {
            steps {
                script {
                    // Vérifier si Poetry est déjà installé
                    def poetryInstalled = sh(
                        script: 'command -v poetry >/dev/null 2>&1',
                        returnStatus: true
                    ) == 0

                    if (!poetryInstalled) {
                        echo "Installation de Poetry..."
                        sh '''
                            # S'assurer que pip est à jour
                            python3 -m pip install --user --upgrade pip

                            # Installer pipx si pas déjà présent
                            python3 -m pip install --user pipx

                            # S'assurer que pipx est dans le PATH
                            python3 -m pipx ensurepath

                            # Installer Poetry via pipx
                            python3 -m pipx install poetry
                        '''
                    } else {
                        echo "Poetry est déjà installé"
                    }

                    // Vérifier la version de Poetry
                    sh 'poetry --version'

                    // Configurer Poetry pour créer les venvs dans le projet
                    sh 'poetry config virtualenvs.in-project true'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Installer les dépendances
                    poetry install --no-dev

                    # Afficher les informations sur l'environnement
                    poetry env info
                '''
            }
        }

        stage('Install Dev Dependencies') {
            when {
                anyOf {
                    branch "develop"
                    branch "feature/*"
                    changeRequest()
                }
            }
            steps {
                sh 'poetry install'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    # Exécuter les tests avec Poetry
                    poetry run pytest --junitxml=test-results.xml --cov=. --cov-report=xml
                '''
            }
            post {
                always {
                    // Publier les résultats des tests si disponibles
                    script {
                        if (fileExists('test-results.xml')) {
                            junit 'test-results.xml'
                        }
                        if (fileExists('coverage.xml')) {
                            publishCoverage adapters: [
                                coberturaAdapter('coverage.xml')
                            ], sourceFileResolver: sourceFiles('STORE_LAST_BUILD')
                        }
                    }
                }
            }
        }

        stage('Lint') {
            when {
                anyOf {
                    branch "develop"
                    branch "feature/*"
                    changeRequest()
                }
            }
            steps {
                sh '''
                    # Linting avec flake8, black, etc. si configurés
                    poetry run flake8 . || true
                    poetry run black --check . || true
                '''
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
                    # Construire le package avec Poetry
                    poetry build

                    # Créer l'archive ZIP si nécessaire
                    zip -r sbdl.zip lib dist/
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'dist/*, sbdl.zip', fingerprint: true
                }
            }
        }

        stage('Release to QA') {
            when {
                branch 'release/*'
            }
            steps {
                sh """
                    scp -i /home/prashant/cred/edge-node_key.pem -o 'StrictHostKeyChecking no' \\
                        -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf \\
                        prashant@40.117.123.105:/home/prashant/sbdl-qa
                """
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    scp -i /home/prashant/cred/edge-node_key.pem -o 'StrictHostKeyChecking no' \\
                        -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf \\
                        prashant@40.117.123.105:/home/prashant/sbdl-prod
                """
            }
        }
    }

    post {
        always {
            // Nettoyer l'espace de travail si nécessaire
            cleanWs()
        }
        failure {
            // Notifications en cas d'échec
            echo "Pipeline failed!"
        }
    }
}