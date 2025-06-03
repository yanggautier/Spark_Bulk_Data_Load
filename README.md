# Spark_Bulk_Data_Load
Un projet issue de cours Udemy https://www.udemy.com/course/apache-spark-programming-in-python-for-beginners,

Il est destiné à lire des données qui se trouvent dans un entrepôt de données issue de 3 tables, puis de traiter ces données et joindre en une seule ligne afin d'envoyer à un topic Kafka pour que les utilisateurs finaux peuvent consommer ces données.


## Spark
Séparer les configurations selon les environnements de configuration

conf/
- sbdl.conf  (configurations des environnement, instances de Kafka)
- spark.conf (configurations de Spark selon l'environnement de travail)

lib/
- ConfigLoader.py (fonctions permettent récupérer les configrations)
- DataLoader.py (fonctions pemettent d'extraire les données)
- logger.py (configuration de Spark logger)
- Transformations.py (fonctions qu'on utilise pour l'étape tranformation)
- Utils.py (fonction pour créer la session Spark)

## CI/CD Avec Jenkins
Configuration avec Github webhook, qui suit des étapes:
- build
- test
- package
- release
- deploy


## Test unitaire
Test avec des données de test
- Test si les données de sources sont lus correctement
- Test si les environnements de développement sont corrects
- La version de Spark si c'est bon
- Test les données de sortie sont correctes




