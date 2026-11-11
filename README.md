# 🗑️ Tounes Propre

**Collecte intelligente des déchets : prioriser les bacs, prédire leur remplissage et optimiser les tournées des camions.**

Projet réalisé pour le **Smart City Tunisia Challenge**, axe **Zero Pollution & a Better Environment**.

- 🔗 Démo en ligne : `<à compléter : lien GitHub Pages>`
- 🎬 Vidéo de démonstration (2 min) : `<à compléter>`
- 📄 Rapport technique : [`docs/rapport.pdf`](docs/) `<à ajouter>`

> ⚠️ Ce dépôt contient un **prototype**. Les niveaux de remplissage sont **simulés** et la carte est schématique.

---

## 1. Le problème

Les camions de collecte suivent souvent des tournées fixes : certains bacs débordent avant leur passage, d’autres sont presque vides quand ils passent. Résultat : carburant gaspillé, émissions inutiles et déchets dans l’espace public.

## 2. La solution

Tounes Propre calcule pour chaque bac un **score de priorité en pourcentage**, prédit quand il sera plein, puis construit des **tournées optimisées** pour 1 à 3 camions en respectant leur capacité. Les citoyens signalent les problèmes sur une page dédiée et suivent leur traitement.

## 3. Pages de l’application

| Page | Accès | Contenu |
|---|---|---|
| 📢 **Citoyen** | Public | Formulaire de signalement : type, zone, photo, position GPS, description. Les doublons sont fusionnés et renforcent la priorité. |
| 📋 **Suivi** | Public | Liste des signalements avec leur statut : Reçu, Repéré, Vidé / traité. |
| 📈 **Analyse** | Public | Tableau de scénarios (distance et CO₂ évité selon le seuil de score), historique du remplissage moyen, export CSV. |
| 🗺️ **Municipalité** | **Privé** (mot de passe) | Carte des bacs, tableau de bord, simulation d’un jour, mesure capteur, calcul des tournées, traitement des signalements. |

### Accès à la page Municipalité

Cliquer sur **« 🔐 Espace agents »** et saisir le mot de passe unique des agents.

- Mot de passe de démonstration : `Tounes2026`
- Il se change dans `index.html`, à la ligne `var MOT_DE_PASSE="Tounes2026";`

> ⚠️ **Limite de sécurité** : le mot de passe est vérifié dans le navigateur. C’est un filtre d’interface, pas une sécurité réelle (il est lisible dans le code). Une version réelle utiliserait une authentification côté serveur.

## 4. Fonctionnalités principales

- **Score de priorité (0 à 100 %)** : niveau de remplissage, signalements ouverts, jours sans collecte, sensibilité du lieu, confirmation par capteur.
- **Prédiction** : « plein dans ≈ X h », à partir de l’historique de chaque bac.
- **Tournées** : 1 à 3 camions, capacité en litres, bacs reportés si le camion est plein, amélioration 2-opt.
- **Distances** : coordonnées approximatives de vrais quartiers du Grand Tunis, distance de Haversine × facteur de détour routier.
- **Signalements** : photo (réduite à 200 px), GPS, fusion des doublons, confirmation par capteur.
- **Capteur** : bouton « Mesure capteur » et fonction `recevoirCapteur(id, niveau)` prête pour une passerelle réelle.
- **Analyse** : scénarios, historique, export CSV.

## 5. Technologies

- **HTML5**, **CSS3** (thème clair et sombre automatique)
- **JavaScript** sans bibliothèque
- Carte en **SVG**
- Stockage : `localStorage` du navigateur

Aucune dépendance, aucun serveur : tout tient dans un seul fichier `index.html`.

## 6. Installation et exécution

### Option A : double-clic (le plus simple)

```bash
git clone https://github.com/<votre-compte>/tounes-propre.git
cd tounes-propre
```
Puis ouvrir `index.html` dans un navigateur (Chrome, Edge, Firefox ou Safari).

### Option B : VS Code

1. Installer l’extension **Live Server**.
2. Clic droit sur `index.html`, puis **Open with Live Server**.

### Option C : serveur local avec Python

```bash
python -m http.server 8000
```
Puis ouvrir <http://localhost:8000>.

### Option D : GitHub Pages (lien public)

1. **Settings → Pages**
2. **Branch** : `main`, dossier `/ (root)`, puis **Save**
3. Le site sera disponible à `https://<votre-compte>.github.io/tounes-propre/`

## 7. Scénario de démonstration

1. **Citoyen** : envoyer un signalement « Bac plein » avec une photo.
2. **Espace agents** : se connecter avec le mot de passe.
3. **Municipalité** : cliquer sur un bac, envoyer une mesure capteur, cliquer sur « Simuler un jour ».
4. Régler le seuil, choisir 2 camions, cliquer sur **Calculer**.
5. Cliquer sur **Collecter**, puis ouvrir **Suivi** (le signalement passe à « Vidé »).
6. Ouvrir **Analyse** : scénarios et historique.

## 8. Comment ça marche

### Score de priorité

```
score (%) = 0,60 × niveau de remplissage (%)
          + 20 × min(signalements ouverts, 3) / 3
          + 10 × min(jours sans collecte, 5) / 5
          +  5 × sensibilité du lieu / 2
          +  5 si un signalement « bac plein » est confirmé (niveau ≥ 70 %)
```

Les poids sont des choix de conception, à calibrer avec les services de collecte.

### Tournées

1. On garde les bacs dont le score dépasse le seuil.
2. Balayage angulaire autour du dépôt pour répartir les bacs entre les camions.
3. Plus proche voisin dans chaque secteur, avec contrainte de capacité (volume d’un bac = 240 L × niveau).
4. Amélioration **2-opt**.
5. Les bacs qui ne rentrent pas sont **reportés**.

### Distances, carburant et CO₂

```
distance = distance de Haversine × facteur de détour
litres   = km × consommation / 100
CO₂ évité (kg) = (km de référence − km optimisés) × consommation / 100 × émission
```

La **référence** est un camion qui visite tous les bacs dans un ordre fixe.

### Hypothèses (estimations à valider)

| Hypothèse | Valeur |
|---|---|
| Facteur de détour routier | 1,3 |
| Consommation du camion | 35 L / 100 km |
| Émission du gasoil | 2,65 kg CO₂ / L |

Ces valeurs sont fixes dans le code et doivent être remplacées par des valeurs sourcées.

## 9. Structure du dépôt

```
tounes-propre/
├── index.html       # application complète (HTML + CSS + JavaScript)
├── README.md        # ce fichier
├── LICENSE
├── .gitignore
└── docs/            # rapport technique, présentation, captures d’écran
```

## 10. Limites

- Données de remplissage **simulées**, sans capteur réel.
- Carte schématique, positions approximatives, distances à vol d’oiseau corrigées par un facteur.
- Heuristiques (plus proche voisin, 2-opt) : rapides, mais pas toujours optimales.
- Données stockées dans le navigateur : pas de partage entre appareils.
- Mot de passe vérifié côté navigateur : filtre d’interface seulement.
- Poids du score et hypothèses de CO₂ non validés sur le terrain.

## 11. Perspectives

- Capteurs à ultrasons (ESP32) envoyant les niveaux en temps réel via une passerelle.
- Vraie carte OpenStreetMap et distances routières.
- Base de données et API avec authentification côté serveur et comptes individuels.
- Application mobile avec photo et GPS natifs.
- Détection automatique du type de déchets par reconnaissance d’image.
- Calibration du score avec des données réelles d’un service de collecte.

## 12. Solutions existantes

Des solutions comme **Clean 8** (2BK Innovation avec la GIZ), **WeClean** ou **Colibris** existent déjà en Tunisie. Tounes Propre en propose une version légère et ouverte, qui combine niveaux de remplissage, signalements citoyens et prédiction, avec des tournées multi-camions sous contrainte de capacité.

## 13. Équipe

| Nom | Rôle |
|---|---|
| `<Prénom Nom>` | `<Développement>` |
| `<Prénom Nom>` | `<Design / Présentation>` |

## 14. Licence

Distribué sous licence MIT. Voir le fichier [LICENSE](LICENSE).
