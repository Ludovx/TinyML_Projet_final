# TinyML_Projet_final

Le but de ce projet est d'utiliser le kit de Tiny Machine Learning d'Arduino et notamment sa caméra pour détecter des cellules cancéreuses sur la peau humaine.

Pour ce faire, nous disposons du matériel suivant :
  - carte Arduino nano 33 BLE Sense Lite
  - caméra OV7675 
  - bouton de commande

Afin de détecter les potentielles cellules cancéreuses, nous allons utiliser un IA entraîner à les détecter sur des images qui seront prises en temps réel avec la caméra OV7675 du kit. Dans un premier temps, pour entraîner et concevoir le modèle de l'IA, nous allons utiliser Edge Impulse. Ensuite, nous allons développer un code Arduino qui tuilise le modèle entraîné pour détecter les anomalies de peau. 

# 1. Création et entraînement du modèle sur Edge Impulse

Dans un premier temps, nous avons essayé de récupérer des images avec la caméra OV7675 pour constituer une base de données pour entraîner le modèle. 

Il s'avère que la focale de la caméra ne permettait pas de prendre des images aussi près de la peau. Nous avons donc utiliser une base de données récupérée sur Kaggle qui a permis d'entraîner le modèle. 

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

Nous allons donc essayer une autre solution. Nous allons générer un firmware à partir de Edge Ipulse que nous allons utiliser pour flasher l'Arduino.


Nous avons décider de fouiller puis d'utiliser les outils mis à notre disposition tel que le EON Tuner qui permet de calculer et comparer différents paramétrages pour le model, dans le cas présent il s'agit d'utiliser des images RGB ou en niveaux de gris, ainsi que d'autres paramètres plus poussés. Nous rentrons les caractéristiques de l'arduino sur le quel le model devra fonctionner ainsi que les limites en temps de calculs, en ram et en rom que l'on désire.
Une fois lancer, EON Tuner nous sort les résultats :
![image](https://user-images.githubusercontent.com/84172586/216776585-a1d8d559-568f-4f2d-85ab-997c0bffd3e6.png)
![image](https://user-images.githubusercontent.com/84172586/216776614-be341f05-0194-4759-bb56-f9cfac294815.png)
![image](https://user-images.githubusercontent.com/84172586/216776663-9d9ee045-fb8b-49b5-b645-d7e862fa0fc6.png)

Parmis les résultats, nous avons décider de ne prendre que ceux avec un taux de réussite de plus de 90%, et nous avons opté pour celui demandant le moins de RAM, afin de ne pas trop solicité l'arduino, qui en plus doit compiler du code et faire fonctioner une camera (nécéssitant beaucoup de RAM).

Avec le model choisi parmis ceux proposé par EON Tuner, nous obtenons ces résultats :
![image](https://user-images.githubusercontent.com/84172586/216776745-298e6654-06aa-4dc0-a09d-6bd013381d3a.png)
![image](https://user-images.githubusercontent.com/84172586/216776755-ae6c5f0b-92c6-49c3-b482-8e9d521cca0f.png)
![image](https://user-images.githubusercontent.com/84172586/216776765-9a2b778f-adbc-4a17-b490-6a17d018942a.png)


Pour tester et pousser le model dans ses limites, nous avons utilisé l'outils de classification en temps réel de EDGE Impulse :
![image](https://user-images.githubusercontent.com/84172586/216776822-ca0c364a-79b0-49a5-9f45-6eca642a39a7.png)
![image](https://user-images.githubusercontent.com/84172586/216776831-966e3c55-6b4e-47df-8df5-c4a0992d4a16.png)

Les résultats et l'interprétation de l'IA ne sont pas concluantes. 









