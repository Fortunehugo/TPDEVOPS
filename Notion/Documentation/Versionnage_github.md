# Versionnage github

Pour initialiser le repository github :  

git init a la racine du projet

git remote add origin git@github.com:NomDuCompte/nomduprojet.git 

git status

git add .

git commit -m "commentaire ou nom du commit"

git push 

git checkout -b développe pour crée la branche développe

modifier un fichier 

git status

git diff 

git commit -m "nom du commit"

git push pour sauvegarder la branche

Pour cloner le projet et l'avoir en local 

![Versionnage_github/Untitled.png](Versionnage_github/Untitled.png)

dans un terminal et un dossier specifique au projet :

git clone git@github.com:NomDuCompte/nomduprojet.git

accepter les demandes du terminal 

SI VOUS N'AVEZ PAS DE CLEF SSH

Dans un terminal , générer une clef ssh : ssh-keygen -t rsa -m PEM

Le terminal vous proposera un chemin par défaut, faite ENTRÉ,

le terminal bous demande de mettre un message, laisser le vide et faite ENTRÉ

Vôtres clef est maintenant créé , placez vous dans le répertoire par défaut indiqué : cd .ssh

normalement deux fichiers id_rsa si trouverons 

faite dans le terminal "cat id_rsa.pub"

copier coller le code et mettez dans vôtres compte github 

![Versionnage_github/Untitled_1.png](Versionnage_github/Untitled_1.png)

![Versionnage_github/Untitled_2.png](Versionnage_github/Untitled_2.png)

![Versionnage_github/Untitled_3.png](Versionnage_github/Untitled_3.png)