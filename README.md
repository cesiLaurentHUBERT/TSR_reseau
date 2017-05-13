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
sudo apt-get update && sudo apt-get install php5 php5-mysql mysql-server mysql-client phpmyadmin

sudo a2enmod php5
```

Et redémarrer apache2

```bash

sudo systemctl restart apache2

```

### Exemples de programmes

#### Connexion

Adaptez ce programme pour vous connecter à votre BDD.

Pour cela, créez un fichier `connexion.php` et modifiez le contenu suivant:

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
Pour cela, créez un fichier `select.php` et modifiez le contenu suivant:

```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";
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

#### Plus d'exemples

Le Net regorge d'exemples de connexion à une BDD en utilisant PHP. [Ce lien](https://www.w3schools.com/php/php_mysql_select.asp) en est un exemple parmi d'autres.

## Protection des données

Créer un répertoire admin dans votre répertoire DocumentRoot.

Y placer un fichier `phpinfo.php` avec le contenu suivant:

```php
<?php phpinfo(); ?>

```

La manipulation pour protéger un dossier web par login et mot de passe sous apache2 est simple.


Il suffit de créer 2 fichiers : le `.htaccess` et `.htpasswd`.

Ces 2 fichiers donnent la méthode d'accès au dossier `.htaccess` et les login et mot de passe `.htpasswd`.

Il est plus que recommandé que ces 2 fichiers commences par les 3 caractères suivant `.ht`.

Ceci permet de les protéger car dans la configuration par défaut d'apache2, les fichiers commençant par ses 3 caractères ne sont pas récupérables grâce au paramétrage suivant de `/etc/apache2/apache2.conf`
```conf
<FilesMatch "^\.ht">
        Require all denied
</FilesMatch>

```

Pour notre exemple nous allons imaginer que notre dossier d'hébergement par défaut est :
`/var/www/apache.exemple.cesi`
et le dossier à protéger est :
`/var/www/apache.exemple.cesi/admin/`

Pour le virtualHost `apache.exemple.cesi`, nous avons la configuration stockée dans `/etc/apache2/sites-available/apache.exemple.cesi`

Son contenu est :
```conf
<VirtualHost *:80>
        DocumentRoot /var/www/apache.exemple.cesi
        ServerName apache.exemple.cesi
        LogLevel warn
</VirtualHost>

```

On se  place dans le dossier que nous voulons protéger et on édite le fichier `.htaccess` :
```bash

cd /var/www/apache.exemple.cesi/admin/

nano .htaccess
```
`

Insérez le code suivant :

```conf

 AuthUserFile /var/www/apache.exemple.cesi/admin/.htpasswd
 AuthName "Acces Limité"
 AuthType Basic
 require valid-user

```


* AuthUserFile indique le fichier contenant les login et mot de passe
* AuthName donne un titre à la fenêtre de mot de passe
* AuthType indique le type d'identification (nous utilisation la méthode la plus simpe : Basic




Créons maintenant le fichier .htpasswd :

```bash
sudo htpasswd -b -c /var/www/apache.exemple.cesi/admin/.htpasswd user motdepasse
```


L'option -b est utilisée pour indiquer le mot de passe directement dans la commande de création du fichier, si on ne la met pas, le mot de passe nous sera demandé juste après
L'option -c permet de créer le fichier .htpasswd
mon_login est le login de votre accès et mon_mot_de_passe est le mot de passe.



Il ne vous reste plus qu'à activer l'option AllowOverride pour votre dossier /var/www/apache.exemple.cesi/admin/.

Pour cela éditez votre fichier de virtualHost et ajoutez-y les lignes suivantes (avant la balise de fermeture) :
<Directory /var/www/apache.exemple.cesi/admin/>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
</Directory>

Votre virtualHost doit donc avoir à peu près cette tête :

```bash
 <VirtualHost *:80>
        DocumentRoot /var/www/apache.exemple.cesi/
        CustomLog /var/log/apache2/apache.exemple.cesi.log
        LogLevel warn

       <Directory /var/www/apache.exemple.cesi/admin/>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>

</VirtualHost>
```

Il ne vous reste plus qu'à redémarrer apache pour mettre à jour votre configuration :
```bash
sudo systemctl restart apache2
```
