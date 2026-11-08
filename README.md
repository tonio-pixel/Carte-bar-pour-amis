# 📍 Meet at the Bar - Carte Interactive

Une application web interactive conçue pour calculer le point de rendez-vous idéal pour un groupe. En saisissant jusqu'à 5 adresses différentes, le système calcule un point de convergence géographique et interroge des bases de données en temps réel pour suggérer de véritables établissements à mi-chemin.

## 🚀 Voir l'application en direct
[👉 Cliquez ici pour ouvrir la carte interactive](https://tonio-pixel.github.io/Carte-bar-pour-amis/)

## ⚙️ Fonctionnalités
* **Géocodage multiple :** Transformation instantanée des adresses postales en coordonnées GPS via Nominatim.
* **Calcul de convergence :** Détermination du centre exact pour minimiser le trajet de chaque participant.
* **Data Réelle (Live) :** Connexion à l'Overpass API (OpenStreetMap) pour extraire les bars et pubs situés dans un rayon de 1 kilomètre autour du point central.
* **Informations ciblées :** Affichage des vrais noms, des horaires d'ouverture officiels, et des données tarifaires/Happy Hour (lorsque les gérants les ont renseignées dans l'Open Data).
* **Interface Dark Mode :** Design UI/UX optimisé pour l'industrie de la restauration et du monde de la nuit.

## 🛠️ Technologies Utilisées
* **Frontend :** HTML5, JavaScript, API Fetch, Tailwind CSS.
* **Cartographie :** Bibliothèque Leaflet.js, fond de carte CartoDB Dark Matter.
* **Open Data :** Nominatim API et Overpass API.
