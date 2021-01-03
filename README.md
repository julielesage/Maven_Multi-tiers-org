# Maven_Multi-tiers-org

Générer un squelette de projet Maven :    
```
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.1
```

Construire un jar :
````
mvn package
````

Installer ce projet :
````
mvn clean install
````

## De la bonne organisation d'un pom

- renseigner informations générales, organization, dépendances, plugins, profiles, build, ...
- entrer la mainClass dans Manifest de l'Archive du maven-jar-plugin
- fichier info.properties dans src/main/resources. ex :   
```
# Propriété contenant la version du projet
org.exemple.demo.version=${project.version}
````
- filtrer les fichiers resources qui n'ont pas à être compilé + info.properties
- déclarer les differents environnements dans les profiles
````
mvn package -Denvironnement=test
````

## Projet Multi-tiers

Plus loin que les patrons MVC et DAO, Maven se découpe en :
batch, webapp, business, consumer (de services), qui ont tous accès au module Model
Pour webapp :
````
mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-webapp 
````
pour les autres : quickstart

**dependencyManagement Dans le POM parent**  
- définir les versions des dépendances
- gérer la transitivité des dépendances : ignorer l'import de certaines sous-dependances si pas besoin avec exclusions
- définir la portée des dépendances dans <scope> avec test, runtime, compile, ou provided
- importer la Bill Of Materials du framework utilisé (spring) <type>pom</type>
  
## Cycles de vie Maven

- default : validate, compile, test, package, install, deploy
- clean : supprime la construction précéddente
- site : génère un site html dans target/staging qui décrit la structure de l'application/ du projet

Les différentes phases des cycles sont gérées par des goals de plugins. On peut les configurer. ex :
```
<plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-compiler-plugin</artifactId>
       <version>3.1</version>
       <configuration>
            <showDeprecation>true</showDeprecation>
            <showWarnings>true</showWarnings>
       </configuration>
</plugin>
````
On peut aussi cabler un goal à l'execution d'une phase. Ex : exécuter enforcer:enforce dans validate pour s'assurer qu'un des profils soit activé

## Packager ses livrables et générer un WAR

- Utilisation de maven-war-plugin pour filtrer les web ressources de la webapp : les jsp header, footer et about de WEB_INF sont inclus dans la config
- S'assurer qu'aucun SNAPSHOT ne soit envoyé en production
- Renseigner les propriétés infos du projet, pour en sortir des variables
- **Générer une archive du jeu de batchs** : créer src/data/scripts et conf dans ticket-batch, configurer le manifest pour maven-jar-plugin,  
créer le fichier src/assembly/archive-deploy.xml, Câbler la génération des fichiers d'archive à la phase package, puis :
````
mvn package
`````
cela crée dans target une archive (bin, conf, lib) tar.gz et zip

## Générer un site sur la structure du projet

src/site/site.xml = site descriptor
Configuration du site dans distributionManagement du pom (url = localhost ou vrai site)
ajouter maven-site-plugin dans le dependencyManagement du POM parent
ajout d'une FAQ ou autre page
demander la génération de rapport de tests pour chaque module dans ReportSets
````
mvn package site site:stage
````
et retrouver l'html dans target/staging




