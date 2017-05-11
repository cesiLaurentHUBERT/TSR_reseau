# TSR_reseau

## Installation
### Linux
En vous basant sur les informations données dans l'article suivant et en utilisant le gestionnaire de paquet `apt-get`, installez un serveur Apache sous Debian

http://siguillaume.developpez.com/tutoriels/apache/installation-configuration-serveur-web-apache/

#### Installation d'un VirtualHost

Un VirtualHost (hôte virtuel) permet de faire fonctionner plusieurs serveurs Web sur une même machine. Celle-ci répondra donc à des requêtes à destinations de plusieurs sites:

* siteA.exemple.com
* www.exemple-deux.com

Chaque hôte virtuel possède son propre répertoire de fonctionnement. Ce répertoire est la racine des documents visibles depuis Internet : `DocumentRoot`

Par exemple:

* /var/www/siteA.exemple
* /var/www/exemple-deux

Ainsi, l'accès à une adresse donnée correspond à un fichier donné:

* `siteA.exemple.com/index.php`
    * `--> /var/www/siteA.exemple/index.php`
* `www.exemple-deux.com/repertoire/fichier.html`
    * `--> /var/www/exemple-deux/repertoire/fichier.html`


#### Configuration
La page suivante donne des exemples de VirtualHost:

http://httpd.apache.org/docs/2.2/fr/vhosts/examples.html


La définition suivante :
```
<VirtualHost *:80>
DocumentRoot /www/example.com
ServerName www.example1.com
</VirtualHost>

```

Indique:

* l'écoute est faite sur le port 80 quel que soit le serveur (`*`)
* la racine des documents est `/www/example.com`
* l'adresse utilisée pour ce VirtualHost est `www.example1.com`. Toute requête commençant par `http://www.example1.com` sera prise en compte ici




### Windows

En utilisant les informations données par l'article précédent, installez un serveur Apache sous Windows.

Vous pouvez également regarder l'utilité de logiciels tels que uWamp, WampServer ou WTServer

## Utilisation

Vous allez configurer votre DNS pour qu'il affiche des sites différents sur les deux machines. A vous de trouver dans la documentation comment on peut faire cela

Pour la machine Linux, configurer deux sites intranet qui pointeront vers cette machine: `bonjour.linux.exemple.cesi` et `test.linux.exemple.cesi`

Pour la machine Windows, configurer deux sites intranet qui pointeront vers cette machine: `bonjour.windows.exemple.cesi` et `test.windows.exemple.cesi`

Les fichiers à placer sont dans le répertoire [contenu](contenu)

## Configuration de PHP et MySQL

### Installation
Il faut d'abord installer le paquet `php5` : 

```bash
sudo apt-get update && sudo apt-get install php5 php5-mysql mysql-server mysql-client

sudo a2enmod php5
```

Et redémarrer apache2

```bash

sudo systemctl restart apache2

```

### Exemples de programmes

#### Connexion

Adaptez ce programme pour vous connecter à votre BDD

```php
 <?php
$servername = "localhost";
$username = "username";
$password = "password";

// Create connection
$conn = new mysqli($servername, $username, $password);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?> 
```

### Récupération de valeur
#### Préparation de la BDD

Créez une table nommée Personne dans une BDD nommée `demophp`

Insérez des éléments en utilisant l'interface phpMyAdmin ou avec la requête suivante:

```sql
INSERT INTO `demophp`.`Personne` (`id`, `nom`, `prenom`) VALUES ('2', 'Assange', 'Julian');
```


#### Programme

```php
<?php
$servername = "lamp.exemple.cesi";
$username = "testphp";
$password = "mdpphp";
$dbname = "demophp";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);
// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$sql = "SELECT id, prenom, nom FROM Personne";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    // output data of each row
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Nom: " . $row["prenom"]. " " . $row["nom"]. "<br>";
    }
} else {
    echo "0 results";
}
$conn->close();
?> 
```

Testez dans votre navigateur


