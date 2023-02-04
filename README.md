# TinyML_Projet_final

Le but de ce projet est d'utiliser le kit de Tiny Machine Learning d'Arduino et notamment sa caméra pour détecter des cellules cancéreuses sur la peau humaine.

Pour ce faire, nous disposons du matériel suivant :
  - carte Arduino nano 33 BLE Sense Lite
  - caméra OV7675 
  - bouton de commande

Afin de détecter les potentielles cellules cancéreuses, nous allons utiliser un IA entraîner à les détecter sur des images qui seront prises en temps réel avec la caméra OV7675 du kit. Dans un premier temps, pour entraîner et concevoir le modèle de l'IA, nous allons utiliser Edge Impulse. Ensuite, nous allons développer un code Arduino qui tuilise le modèle entraîné pour détecter les anomalies de peau. 

# 1. Création et entraînement du modèle sur Edge Impulse

Dans un premier temps, nous avons essayé de récupérer des images avec la caméra OV7675 pour constituer une base de données pour entraîner le modèle. 

Il s'avère que la focale de la caméra ne permettait pas de prendre des images aussi près de la peau :
![image](https://user-images.githubusercontent.com/92917769/216774862-c42121de-5788-42d9-b48d-96cd29c23446.png)

Nous avons donc utiliser une base de données récupérée sur Kaggle qui a permis d'entraîner le modèle. 

Les caratéristiques de l'entraînement sont les suivantes :
![caracteristic_model](https://user-images.githubusercontent.com/92917769/216769477-3fe07546-200b-4d78-bdca-8b7939eaf94a.png)

Au final, nous avons obtenu les résultat suivant sur l'entraînement :
![model](https://user-images.githubusercontent.com/92917769/216769511-690fe6a6-b810-4c4e-a414-2a228a695e1c.png)

Et voici les résultats que nous avons obtenu pour les tests du modèle :
![model_testing](https://user-images.githubusercontent.com/92917769/216769531-596bd68f-685f-4196-8c71-7e79d1011971.png)

# 2. Création du code Arduino pour utiliser le modèle

Une fois le modèle prêt, nous avons télécharger les fichiers générés par Edge Impulse comprenant le modèle, les librairies et quelques exemples d'utilisation :

![image](https://user-images.githubusercontent.com/92917769/216769650-c92924aa-0344-4cad-8bd7-8f768c2cec6d.png)

En cherchant dans les fichiers, nous avons donc voulu tester l'exemple "nano_ble33_sense_camera" :
![image](https://user-images.githubusercontent.com/92917769/216769710-a6ccf9ce-5d42-4f7a-894c-deabd08ee832.png)

Lors de l'exécution de ce code, nous avons tout de suite été confronté à des erreurs empéchant d'arriver à un résultat correct :
![Errors_](https://user-images.githubusercontent.com/92917769/216769761-edcdf598-698a-40a8-86de-51deb72f16f4.png)

Comme on peut l'observer, il semble y avoir des erreurs d'allocation mémoire sur la carte Arduino qui empèche le code de s'exécuter correctement.
En cherchant dans le code, on s'est aperçu que l'éditeur avait proposé une correction d'erreur en commentaires :
![Errors_explication](https://user-images.githubusercontent.com/92917769/216769805-7b74a224-d1e3-4f99-9f19-297a262caf82.png)

Nous avons donc suivi ces instrutions mais malheureusement, les erreurs ont persisté et nous n'avons pas pu avoir de résultat. 
D'après ce commentaire, si les erreurs persistent, c'est que notre carte ne possède pas assez de mémoire. Or cela parâit étrange car le code est censé être conçu justement pour notre carte. 

Après cela, nous vons décidé de vérifier si les pins de la caméra attribués dans la librairie du modèle était les même que ceux de notre kit. 
Nous avons donc été chercher la déclaration des pins dans les librairies :

![Pin_attribution_camera_OV7675](https://user-images.githubusercontent.com/92917769/216770112-03d9ca19-9d1b-4a78-a54b-54ff4d762e6d.png)

Et nous nous sommes procuré le schéma électronique de la carte PCB du kit sur laquelle est branché l'Arduino :
![image](https://user-images.githubusercontent.com/92917769/216770107-17d380ce-25a3-4d84-8c0f-be4a2bd531ad.png)

En comparant les deux documents, on s'aperçoit que les pins ne correspondent par forcément et ont des noms différents.
En cherchant la documentation précise de la caméra, nous n'avons trouvé que celle d'un autre modèle dont nous avons pu analyser le schéma électronique (OV2640) :

![image](https://user-images.githubusercontent.com/92917769/216770815-72e9a58c-278e-4700-8328-250d5c35f114.png)

En faisant l'hypothèse que cette configuration est aussi celle de notre caméra OV7675, alors les pins sont bien définis dans la librairie et le problème ne vient pas de là. 

Nous avons ensuite décidé de refaire un modèle plus léger pour tenter de l'utiliser sur l'Arduino. 
Nous avons donc eu l'idée de convertir les images en niveaux de gris :

![image](https://user-images.githubusercontent.com/92917769/216774946-05acd709-8191-4a7a-a786-f4945f30275b.png)

Nous avons obtenu les résultats suivants :

![image](https://user-images.githubusercontent.com/92917769/216774993-23ff1bf7-6eee-426f-bd4a-80a540b62002.png)
![image](https://user-images.githubusercontent.com/92917769/216775014-ca4409df-2a65-4277-b703-99cde435aa7c.png)

En essayant d'utiliser le nouveau modèle sur l'Arduino, nous avons obtenu les mêmes erreurs que pour les modèles précédents. 

Nous avons quand même voulu effectuer des tests pour savoir si le modèle était fonctionnel. Pour cela nous avons upload des images sur Edge Impulse prises avec la caméra pour les tester.

La première image est une image de psoriasis. Le modèle a bien répondu :

![image](https://user-images.githubusercontent.com/92917769/216775292-3605b430-74ec-430d-8950-ed00da17a413.png)

La deuxième image est une image de grain de beauté. Le modèle a bien répondu :

![image](https://user-images.githubusercontent.com/92917769/216775381-1fa782c0-8075-4491-b22c-24766b8edee9.png)

La deuxième image est une image de peau normale. Le modèle a bien répondu :



Suite à ce nouvel échec, nous avons décidé de demander à Edge Impulse d'analyser le meilleur modèle pour notre Arduino :

![image](https://user-images.githubusercontent.com/92917769/216775491-a63ff503-47ab-4437-923e-d502d39ab4c2.png)

Nous avons donc sélectionner le modèle marqué avec l'étoile jaune et cela nous a donné les résultats suivants :

![image](https://user-images.githubusercontent.com/92917769/216775535-d1e4b4fd-0f26-4cfd-a1de-beb02a8f2889.png)
![image](https://user-images.githubusercontent.com/92917769/216775551-51f30b01-1d21-4597-ab0b-69ab0ff68a31.png)
![image](https://user-images.githubusercontent.com/92917769/216775558-3efd1ec9-686a-4644-a174-fb3aceb97516.png)

