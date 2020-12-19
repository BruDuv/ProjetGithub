# Tutoriel Postgresql / PHP / Leaflet


## Récupérer des données depuis leaflet :

Fichier php

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

Fichier js

    //création de la variable map
    var map = L.map('map');
    //appel osm//
    var osmUrl='http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png';
    //attribution osm//
    var osmAttrib='Map data © OpenStreetMap contributors';
    //création de la couche osm//
    var osm = new L.TileLayer(osmUrl, {attribution: osmAttrib}).addTo(map);
    //centrage de la carte//
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
