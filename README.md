# Carte-bar-pour-amis
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Point de Rendez-vous Bars</title>
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        #map { height: 100%; min-height: 400px; z-index: 10; }
        .leaflet-popup-content-wrapper { background-color: #1f2937; color: #f3f4f6; border-radius: 0.5rem; }
        .leaflet-popup-tip { background-color: #1f2937; }
    </style>
</head>
<body class="bg-gray-900 text-gray-100 flex flex-col md:flex-row h-screen font-sans">

    <!-- Panneau Latéral (Contrôles) -->
    <div class="w-full md:w-1/3 p-6 bg-gray-800 shadow-xl overflow-y-auto flex flex-col gap-4 z-20">
        <h1 class="text-2xl font-bold text-pink-500 mb-2">📍 Meet at the Bar</h1>
        <p class="text-sm text-gray-400 mb-4">Saisissez jusqu'à 5 adresses pour trouver le bar idéal à mi-chemin.</p>
        
        <div id="address-inputs" class="flex flex-col gap-3">
            <input type="text" placeholder="Adresse 1 (ex: Gare de Lyon, Paris)" class="p-3 bg-gray-700 rounded-lg border border-gray-600 focus:outline-none focus:border-pink-500" value="Gare de Lyon, Paris">
            <input type="text" placeholder="Adresse 2 (ex: Bastille, Paris)" class="p-3 bg-gray-700 rounded-lg border border-gray-600 focus:outline-none focus:border-pink-500" value="Châtelet, Paris">
            <input type="text" placeholder="Adresse 3 (optionnelle)" class="p-3 bg-gray-700 rounded-lg border border-gray-600 focus:outline-none focus:border-pink-500">
            <input type="text" placeholder="Adresse 4 (optionnelle)" class="p-3 bg-gray-700 rounded-lg border border-gray-600 focus:outline-none focus:border-pink-500">
            <input type="text" placeholder="Adresse 5 (optionnelle)" class="p-3 bg-gray-700 rounded-lg border border-gray-600 focus:outline-none focus:border-pink-500">
        </div>

        <button id="calculate-btn" class="mt-4 bg-pink-600 hover:bg-pink-700 text-white font-bold py-3 px-4 rounded-lg transition-colors">
            Calculer le point de rendez-vous
        </button>
        <div id="status" class="text-sm text-pink-400 mt-2 hidden">Calcul et géocodage en cours...</div>
    </div>

    <!-- Carte -->
    <div class="w-full md:w-2/3 h-full relative">
        <div id="map"></div>
    </div>

    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        // Initialisation de la carte (centrée sur Paris par défaut)
        const map = L.map('map').setView([48.8566, 2.3522], 13);

        // FOND DE CARTE GRATUIT (CartoDB Dark Matter) - Aucune clé API n'est requise
        L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
            subdomains: 'abcd',
            maxZoom: 20
        }).addTo(map);

        // Groupes de marqueurs pour pouvoir les effacer lors d'une nouvelle recherche
        let userMarkers = L.layerGroup().addTo(map);
        let barMarkers = L.layerGroup().addTo(map);

        // Icônes personnalisées
        const userIcon = L.icon({
            iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-blue.png',
            iconSize: [25, 41], iconAnchor: [12, 41], popupAnchor: [1, -34]
        });
        const barIcon = L.icon({
            iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-gold.png',
            iconSize: [25, 41], iconAnchor: [12, 41], popupAnchor: [1, -34]
        });

        // Fonction pour géocoder via l'API Nominatim (OpenStreetMap)
        async function geocode(address) {
            const url = `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(address)}&limit=1`;
            const response = await fetch(url);
            const data = await response.json();
            if (data && data.length > 0) {
                return { lat: parseFloat(data[0].lat), lon: parseFloat(data[0].lon) };
            }
            return null;
        }

        // Données simulées de bars (pour démontrer les pop-ups)
        const mockBars = [
            { nom: "Le Pixel", type: "Bar d'application & Mixologie", prix: "4.50€", event: "Soirée jeux de plateau de stratégie & Création de mocktails", latOffset: 0.002, lonOffset: 0.002 },
            { nom: "Oroute / Atelier B", type: "Foodtruck sédentarisé (Spécialité Belge)", prix: "6.00€", event: "Dégustation : Boulets liégeois au blanc de bœuf", latOffset: -0.001, lonOffset: 0.003 },
            { nom: "La Mine", type: "Pub atypique", prix: "7.00€", event: "Exposition de minéraux rares et musique live", latOffset: 0.003, lonOffset: -0.002 },
            { nom: "Le QG du Campus", type: "Brasserie classique", prix: "5.50€", event: "Happy Hour prolongé", latOffset: -0.002, lonOffset: -0.001 }
        ];

        document.getElementById('calculate-btn').addEventListener('click', async () => {
            const inputs = document.querySelectorAll('#address-inputs input');
            const status = document.getElementById('status');
            status.classList.remove('hidden');
            
            userMarkers.clearLayers();
            barMarkers.clearLayers();
            
            let coords = [];
            
            // Récupération des adresses (séquentielle pour ne pas saturer l'API publique)
            for (let input of inputs) {
                if (input.value.trim() !== "") {
                    let coord = await geocode(input.value);
                    if (coord) {
                        coords.push(coord);
                        L.marker([coord.lat, coord.lon], {icon: userIcon})
                            .bindPopup(`<b>Départ :</b> ${input.value}`)
                            .addTo(userMarkers);
                    }
                }
                // Pause de 500ms exigée par les conditions d'utilisation de Nominatim
                await new Promise(r => setTimeout(r, 500));
            }

            if (coords.length > 0) {
                // Calcul du centroïde (milieu géographique exact)
                let avgLat = coords.reduce((sum, c) => sum + c.lat, 0) / coords.length;
                let avgLon = coords.reduce((sum, c) => sum + c.lon, 0) / coords.length;

                map.setView([avgLat, avgLon], 15);
                
                // Ajout des bars simulés autour du centre calculé
                mockBars.forEach(bar => {
                    let barLat = avgLat + bar.latOffset;
                    let barLon = avgLon + bar.lonOffset;
                    
                    let popupContent = `
                        <div class="text-sm min-w-[200px]">
                            <h3 class="font-bold text-pink-400 text-base border-b border-gray-600 pb-1 mb-2">${bar.nom}</h3>
                            <p class="mb-1"><b>Type :</b> ${bar.type}</p>
                            <p class="mb-1"><b>Prix pinte :</b> ${bar.prix}</p>
                            <p class="text-pink-300 mt-2">✨ <b>En ce moment :</b> ${bar.event}</p>
                        </div>
                    `;
                    
                    L.marker([barLat, barLon], {icon: barIcon})
                        .bindPopup(popupContent)
                        .addTo(barMarkers);
                });
            } else {
                alert("Impossible de localiser les adresses. Veuillez essayer avec des noms de rue et de ville plus précis.");
            }
            
            status.classList.add('hidden');
        });
    </script>
</body>
</html>
