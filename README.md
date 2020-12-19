# Tutoriel Postgresql / PHP / Leaflet


## Récupérer des données depuis leaflet :

Dans ce tutoriel, nous allons récupérer l'emplacement de point lors d'un clic avec la souris. 


#### Fichier php
C'est dans le fichier php que la connexion à la base de données se fait, puis la récupération des variables et enfin le remplissage de la base de données.

    //Connexion à la base de donnée postgresql :
    $dbconn = pg_connect('host=localhost dbname=votre_base_de_donnee user=votre_user password=votre_mdp')
    or die('Could not connect: ' . pg_last_error());

    //Récupération des variables lat et longitude :
    $lat = $_POST['lat'];
    $lng = $_POST['lng'];

    //Définition de la requête d'insertion des points :
    $query = "INSERT INTO point (geom) VALUES (ST_SetSRID( ST_Point( " . strval($lng) . ", " . strval($lat) . "), 4326));";

    //lancement de la requête :
    $result = pg_query($query) or die('Query failed: ' . pg_last_error());

    //affichage dans le navigateur dans l'onglet réseau :
    echo = $query

#### Fichier js
Dans le fichier javascript, il s'agit de récupérer les coordonnées lors d'un clic sur la carte puis de les envoyer via une requête xhr.
>Les objets XMLHttpRequest (XHR) permettent d'interagir avec des serveurs. On peut récupérer des données à partir d'une URL sans avoir à rafraîchir complètement la page. Cela permet à une page web d'être mise à jour sans perturber les actions de l'utilisateur. XMLHttpRequest est beaucoup utilisé par l'approche AJAX.

Source : [MDN Web Docs](https://developer.mozilla.org/fr/docs/Web/API/XMLHttpRequest)

    //création de la variable map
    var map = L.map('map');
    //appel osm
    var osmUrl='http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png';
    //attribution osm
    var osmAttrib='Map data © OpenStreetMap contributors';
    //création de la couche osm
    var osm = new L.TileLayer(osmUrl, {attribution: osmAttrib}).addTo(map);
    //centrage de la carte
    map.setView([45.7175, 4.919], 9);

    //Récupération des coordonnées du clic de la carte
    map.on('click', function(e){
        lat = e.latlng.lat;
        lng = e.latlng.lng;
        //envoi de la fonction
        launchXHR(lat, lng)
    })
    
    //fonction qui envoie les données récupérées dans la base de données
    function launchXHR(lat, lng) {
        //création de la requête         
        var xhttp = new XMLHttpRequest();  
        //requête sur le fichier php avec la méthode post
        xhttp.open("POST", "php/test.php", true);
        //
        xhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
        //envoie des données
        xhttp.send("lat="+lat+"&lng="+lng);
    }

#### Fichier Html

    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8" />
            <title>Titre</title>
            <link rel="icon" href="tal.ico"/>
            <link rel="stylesheet" href="style.css" />
            <script src="leaflet/leaflet.js"></script>
            <link rel="stylesheet" href="leaflet/leaflet.css"/>
        </head>
        <body>            
            <div id='map'></div>                    
        </body>
        <script src="script.js"></script>
    </html>

#### Fichier css

    body {
        background-color: rgb(255, 40, 40)
    }
    h1 {
        font-size: 15px
    }
    #map{
        width:500px;
        height:500px;
    }

## Afficher des données depuis postgresql :

#### Fichier php

    <?php
    //Connexion à la base de donnée postgresql :
    $dbconn = pg_connect("host=localhost port=5432 dbname=asso_test_2 user=postgres password=Romainduris")
    or die('Could not connect: '. preg_last_error());
    
    //Création de la requête de sélection des données avec une transformation dans l'EPSG 4326 et ajout d'un champs geojson à partir du st_AsGeojson
    $query = "SELECT * ,st_AsGeojson(ST_Transform(geom, 4326)) as geojson from commune;";
    //lancement de la requête
    $result = pg_query($query) or die('Query failed: ' . pg_last_error());
    //envoie des données dans un tableau :
    $array = pg_fetch_all($result);
    //renvoi des données
    echo json_encode($array);
    ?>

#### Fichier JS

    //création de la variable map
    var map = L.map('map');
    //appel osm
    var osmUrl='http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png';
    //attribution osm
    var osmAttrib='Map data © OpenStreetMap contributors';
    //création de la couche osm
    var osm = new L.TileLayer(osmUrl, {attribution: osmAttrib}).addTo(map);
    //centrage de la carte
    map.setView([45.7175, 4.919], 9);

    // Appel du script php
    //style pour la couche commune
    var style_commune = {
        "color": "#fff",
        "weight": 2,
        "opacity": 0.65
    };

    //Appel de la couche commune
    var xhttp = new XMLHttpRequest();
    xhttp.onreadystatechange = function() {
        //lecture de la connexion au fichier php
        if (this.readyState == 4 && this.status ==200) {
            //récupération du résultat de la requête sql et parcours de la couche :   
            let response = JSON.parse(xhttp.responseText)            
            //transformation du tableau récupéré en couche geojson
            response.forEach((el) => {
                L.geoJSON(JSON.parse(el.geojson),{
                //application du style
                style: myStyle,
                onEachFeature : onEachFeature
                }).addTo(map)
                })
        }
        };
    //requête du fichier php
    xhttp.open("GET", "php/commune.php",true);
    //envoie de la commande au fichier
    xhttp.send();

#### Fichier HMTL 

    <! DOCTYPE html>
    <html>
        <head>
            <meta charset = "utf-8" />
            <title> Titre </title>
            <script src="leaflet/leaflet.js"></script>      
            <link rel = "stylesheet" href="leaflet/leaflet.css" />
            <link rel = "stylesheet" href="css/style.css" />
        </head>    
        <body>
            <h1> La première carte c'est la première carte </h1>        
            <div id = "map"></div>
                    
        </body>
        <script src ="js/script.js"></script>
    </html>


### Bibliographie

Requête [XMLHttpRequest (XHR)](https://developer.mozilla.org/fr/docs/Web/API/XMLHttpRequest)
