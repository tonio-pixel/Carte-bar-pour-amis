# 🍻 Meet at the Bar

> Trouvez le bar parfait au point d'équidistance de votre groupe d'amis. 

## 📖 Description

**Meet at the Bar** est une application web single-page qui permet à un groupe de 2 à 5 personnes de saisir leurs adresses de départ et leur moyen de transport respectif. L'application calcule un **point de rendez-vous équitable** (barycentre pondéré) et affiche les **3 meilleurs bars** les plus proches de ce point, directement sur une carte interactive en mode sombre.

## ✨ Fonctionnalités

- ✅ **Saisie jusqu'à 5 participants** avec adresse et moyen de transport
- ✅ **Géocodage** des adresses via Nominatim (OpenStreetMap)
- ✅ **Barycentre pondéré** : le centre s'adapte au moyen de transport de chacun
- ✅ **Recherche de bars réels** via l'Overpass API (OpenStreetMap)
- ✅ **Top 3 strict** : uniquement les 3 bars les plus proches du point de RDV
- ✅ **Popups détaillés** : nom, type, horaires, happy hour, prix
- ✅ **Dark mode natif** sans clé API (filtre CSS sur les tuiles OSM)
- ✅ **100% gratuit** — aucune clé API requise
- ✅ **Responsive** — fonctionne sur mobile et desktop

## 🧮 Logique de calcul

### Barycentre pondéré

Chaque moyen de transport se voit attribuer un **poids mathématique** :

| Transport | Icône | Poids | Effet |
|-----------|-------|-------|-------|
| À pied | 🚶 | 4.0 | Attire fortement le centre vers soi |
| Métro/Bus | 🚇 | 2.0 | Attrait modéré |
| Vélo | 🚲 | 1.5 | Attrait faible |
| Voiture | 🚗 | 1.0 | S'éloigne le plus du centre |

> **Principe** : quelqu'un à pied accepte de marcher plus loin, donc le centre se rapproche de lui. Quelqu'un en voiture peut parcourir plus de distance, donc le centre s'en éloigne.

### Formule

$$\text{Centre} = \frac{\sum_{i=1}^{n}(\text{position}_i \times \text{poids}_i)}{\sum_{i=1}^{n} \text{poids}_i}$$

### Filtrage des bars

```javascript
bars.sort((a, b) => a.distance - b.distance).slice(0, 3);
