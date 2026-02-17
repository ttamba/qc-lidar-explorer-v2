# QC LiDAR Explorer (V1) – prêt à déployer

V1 = appli MapLibre + recherche par emprise (GeoJSON d’empreintes) + chargement LiDAR COPC/EPT via `maplibre-gl-lidar`.

- Lib LiDAR/streaming: `maplibre-gl-lidar` (OpenGeos) – COPC (streaming octree) + EPT (ept.json).  
- L’inspiration “USGS-like” est le pattern de contrôle/panneau + footprints + streaming.

## Pré-requis
- Node 18+ (recommandé: Node 20)

## Lancer en dev
```bash
npm install
npm run dev
```
Puis ouvrir l’URL affichée (par défaut http://localhost:5173).

## Build + preview
```bash
npm run build
npm run preview
```

## Déploiement Docker (Nginx)
```bash
docker build -t qc-lidar-explorer:v1 .
docker run --rm -p 8080:80 qc-lidar-explorer:v1
```
Puis ouvrir http://localhost:8080

---

## Remplacer les données “démo” par les données Québec

### 1) Empreintes (index des tuiles)
V1 charge `public/tiles.sample.geojson`.  
Tu dois le remplacer par un GeoJSON généré depuis l’index officiel (GPKG) publié sur Données Québec.

Pipeline type (local):
1. Télécharger le ZIP d’index (GPKG) depuis Données Québec
2. Extraire le GPKG
3. Convertir en GeoJSON:
```bash
ogrinfo Index_lidar.gpkg
ogr2ogr -f GeoJSON public/tiles.qc.geojson Index_lidar.gpkg <NOM_DE_LA_COUCHE>
```
4. Modifier `src/main.ts` pour charger `tiles.qc.geojson`.

### 2) URLs COPC/EPT (streaming web)
Pour l’expérience “USGS-like”, les propriétés de chaque feature doivent contenir:
- `copc_url` vers un fichier `.copc.laz` (HTTP + CORS + support range requests)
OU
- `ept_url` vers un `ept.json` (dossier EPT public)

Le portail Québec distribue souvent du LAZ “classique”. Pour streamer:
- Convertir en **COPC** (recommandé) ou générer un **EPT** (Entwine), puis héberger.

---

## Champs attendus dans le GeoJSON
Dans chaque feature:
```json
{
  "properties": {
    "id": "string",
    "name": "string",
    "copc_url": "https://.../*.copc.laz",
    "ept_url": "https://.../ept.json",
    "download_url": "https://... (optionnel)"
  }
}
```

---

## Notes
- Le contrôle LiDAR complet (palette, point size, classes, pick/tooltip) est fourni par `maplibre-gl-lidar`.
- V1 ajoute juste le panneau “QC” pour la recherche/footprints et déclenche `lidarControl.loadPointCloud(url)`.



## Changelog
- 0.1.1: Fix TypeScript build (remove usage of LidarControl.once, use setTimeout flyToPointCloud)
