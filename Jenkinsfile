pipeline {
    agent any

    stages {
        stage('Setup Poetry') {
            steps {
                // S'assurer que Python 3 est disponible et installer pipx
                // pipx est l'outil recommandé pour installer des applications Python dans des environnements isolés
                sh '''
                    python3 -m pip install --user pipx
                    # Ajouter le répertoire des exécutables de pipx au PATH pour que 'pipx' soit trouvable
                    export PATH="$HOME/.local/bin:$PATH"
                    # Installer poetry via pipx. --force assure qu'il est installé/mis à jour.
                    # Utiliser python3 -m pipx pour s'assurer que pipx est appelé correctement après son installation
                    python3 -m pipx install poetry --force
                    # Vérifier que poetry est maintenant disponible
                    poetry --version
                '''
            }
        }
        stage('Build') {
            steps {
               // Maintenant que poetry est installé et dans le PATH, vous pouvez l'utiliser
               sh 'poetry install'
            }
        }
        stage('Test') {
            steps {
               sh 'poetry run pytest'
            }
        }
        stage('Package') {
        when{
           anyOf{ branch "main" ; branch 'release' }
        }
            steps {
               sh 'zip -r sbdl.zip lib'
            }
        }
    stage('Release') {
       when{
          branch 'release'
       }
           steps {
              sh "scp -i /home/prashant/cred/edge-node_key.pem -o 'StrictHostKeyChecking no' -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf prashant@40.117.123.105:/home/prashant/sbdl-qa"
           }
        }
    stage('Deploy') {
       when{
          branch 'main'
       }
           steps {
               sh "scp -i /home/prashant/cred/edge-node_key.pem -o 'StrictHostKeyChecking no' -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf prashant@40.117.123.105:/home/prashant/sbdl-prod"
           }
        }
    }
}