# Connecter AWS à Heroku

[https://youtu.be/_fHp8_9mfGg](https://youtu.be/_fHp8_9mfGg)

### **1 - CRÉER AWS RELATIONAL DATA SERVICE (RDS)**

Dans votre compte AWS, accédez à Bases de données -> RDS

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/rds.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/rds.jpg)

Sélectionnez ensuite «Créer une base de données» et sélectionnez votre type de base de données. Nous utiliserons «PostgreSQL» pour cet exemple.

Allez-y et remplissez le formulaire. Il faut ensuite donner à votre instance de base de données un nom , un nom d' utilisateur et un mot de passe .

Dans la page suivante du formulaire, vous pouvez laisser la plupart des champs par défaut, sauf si vous avez une préférence spécifique. Le principal ici est de donner un nom à votre base de données.

Une fois que vous avez soumis le formulaire, votre base de données doit être créée et vous pouvez maintenant afficher votre base de données à partir de l'onglet instances

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/instances.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/instances.jpg)

### **2 - AJOUTEZ NOTRE AWS RDS À HEROKU**

Par défaut, Heroku créera généralement une nouvelle base de données pour vous. Nous devrons supprimer cette base de données et ajouter dans notre formulaire de base de données AWS.

Supprimons d'abord notre base de données. Pour supprimer votre base de données existante, allez projeter dans heroku, puis «Ressources». Si vous avez une base de données, vous la verrez dans «Addons». Vous pourrez le supprimer à partir de là. 

Maintenant  nous avons supprimé notre ancienne base de données, nous pouvons maintenant définir manuellement la variable de configuration.

Créez un appel variable DATABASE_URL et ajoutez ce qui suit:

"Mysql2: // *nom d'utilisateur* : *mot de passe* @ nom d'hôte / nom de base de données? Sslca = config / amazon-rds-ca-cert.pem"

Min ressemble à ceci:

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/config-var.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/config-var.jpg)

" Postgresql: // ivanov11: ** PASSWORD! @ Lims10dbinstance.ctlf **** b5ug.us-west-2.rds.amazonaws.com/lims10db "

Cela pointe maintenant Heroku vers votre base de données Amazon. Il ne nous reste plus qu'à retourner dans notre base de données Amazon et configurer le groupe SSL et sécurité

Vous pouvez récupérer le dernier élément après le mot de passe dans votre instance de base de données sous "Endpoint"

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/endpoint.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/endpoint.jpg)

### **3 - AUTORISER L'ACCÈS HEROKU À L'INSTANCE RDS**

[Documentation Heroku](https://devcenter.heroku.com/articles/amazon-rds)

Configurer l'instance RDS pour n'accepter que les connexions cryptées SSL des utilisateurs autorisés

Depuis votre tableau de bord RDS, allez dans «Groupes de paramètres» et cliquez sur «Créer un groupe de paramètres».

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/parameter-groups.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/parameter-groups.jpg)

Allez-y et créez un groupe de paramètres. Une fois que vous avez créé votre groupe de paramètres, cliquez dessus et recherchez «rds.force_ssll».

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/parameter-groups-1.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/parameter-groups-1.jpg)

Modifiez la valeur de rds.force_ssl et définissez la valeur sur «1» et enregistrez les modifications.

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/parameter-groups-2.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/parameter-groups-2.jpg)

Configurez RDS le groupe de sécurité pour autoriser toutes les plages d'adresses IP entrantes

À partir de votre instance, sous Détails → Sécurité et réseau, recherchez et cliquez sur les groupes de sécurité. Cela vous mènera à la console de gestion EC2.

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/security-groups.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/security-groups.jpg)

Assurez-vous que votre ID de groupe est sélectionné et choisissez Inbound, puis "Modifier"

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/inbound.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/inbound.jpg)

Changez la source n'importe où et enregistrez.

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/anywhere-1.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/anywhere-1.jpg)

### **4 - OUVRIR LA BASE DE DONNÉES DANS POSTGRESQL**

Pour afficher votre base de données, assurez-vous que Postgres est installé localement.

Commencez par créer un nouveau serveur. Une fois que nous avons nommé notre serveur, nous devons accéder à la connexion et utiliser nos informations d'identification AWS RDS pour nous connecter.

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/create-database-aws-1.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/create-database-aws-1.jpg)

Notre nom d'hôte est la même instance de base de données que nous avons utilisée pour nous connecter dans heroku.

Utilisez le mot de passe et le nom d'utilisateur que vous avez créés précédemment lors de la création de notre base de données sur AWS

### **5 - POUSSER LE PROJET VERS HEROKU**

Si vous l'avez déjà fait, nous devons nous assurer de pousser la dernière version de notre projet pour nous assurer que notre «certificat racine SSL AWS RDS» que nous avons ajouté à l'étape 2 se trouve sur notre serveur en direct.

### **TÉLÉCHARGEMENT ET INSTALLATION DU CERTIFICAT RACINE SSL AWS RDS**

Ce fichier permet de crypter la **connexion à notre base de données** . Vous pouvez trouver ce fichier en effectuant une simple recherche Google « [Certificat racine SSL AWS RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html) » ou en accédant directement au lien. Choisissez le lien qui décrit votre base de données. J'ai utilisé le lien disponible pour toutes les instances de bases de données sous la liste.

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/pem.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/pem.jpg)

Cliquez sur l'un des liens pour télécharger le fichier suivant.

![https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/pem1.jpg](https://secureservercdn.net/45.40.144.200/tm3.3fa.myftpupload.com/wp-content/uploads/2018/09/pem1.jpg)