# Stikstofafstand — publieke starter

Publieksgerichte Next.js + PostGIS webapp om vanaf een Nederlands adres de kortste hemelsbrede afstand te berekenen tot het dichtstbijzijnde **relevante stikstofgevoelige AERIUS-hexagoon van 1 hectare**.

## Wat is toegevoegd

- Adres-autocomplete via de PDOK Location API.
- Exacte geocoding via PDOK, server-side.
- PostGIS KNN-nearest-neighbour query in EPSG:28992.
- Afstand tot de **rand** van het hexagoon met `ST_Distance`.
- `ST_ClosestPoint` voor een visueel controleerbare afstandslijn.
- MapLibre-kaart met OpenStreetMap-basemap, adrespunt, hexagoon en afstandslijn.
- AERIUS habitatcodes en minimale KDW in het resultaat.
- Release-metadata in de database.
- Docker Compose voor lokale PostGIS.
- Import- en update-scripts voor AERIUS GeoPackages.
- Privacy- en juridische disclaimer in de UI.

## Databronnen

AERIUS/RIVM publiceert een koppeltabel tussen het hexagonengrid en relevante stikstofgevoelige habitattypen. De webservice bevat het fijnste resolutieniveau van 1 hectare (zoomlevel 1). Voor geometrie wordt het officiële AERIUS hexagonengrid gebruikt.

PDOK/Kadaster levert de Location API voor Nederlands adreszoeken en geocoding. Deze is publiek, zonder authenticatie en wordt dagelijks bijgewerkt.

## Lokale installatie

```bash
docker compose up -d
cp .env.example .env.local
npm install
npm run dev
```

Daarna moeten de AERIUS-data worden geïmporteerd.

## AERIUS import

Vereisten: GDAL (`ogr2ogr`/`ogrinfo`), `psql`, unzip/curl en toegang tot de PostGIS-database.

Download zowel:
1. de actuele AERIUS **hexagonengrid** GeoPackage;
2. de actuele AERIUS **koppeltabel hexagonengrid en relevante habitats** GeoPackage.

Importeer:

```bash
export DATABASE_URL='postgresql://stikstof:stikstof@localhost:5432/stikstof'
export OGR_PG='PG:host=localhost port=5432 dbname=stikstof user=stikstof password=stikstof'
export AERIUS_RELEASE='AERIUS 2025 / release-datum'
./scripts/import-aerius.sh /pad/hexagons.gpkg /pad/relevante-habitats.gpkg
```

Voor geautomatiseerde updates kun je de officiële ZIP-download-URL's als environment variables meegeven:

```bash
AERIUS_HEX_ZIP_URL='https://officiele-url/hexagons.zip' \
AERIUS_HAB_ZIP_URL='https://officiele-url/relevante-habitats.zip' \
DATABASE_URL='...' OGR_PG='PG:...' AERIUS_RELEASE='AERIUS 2025' \
./scripts/update-aerius.sh
```

De URL's zijn bewust niet hard-coded: RIVM kan bestandsnamen per release wijzigen. Zo faalt een oude URL niet stilletjes terwijl de site een verouderde dataset blijft tonen.

## Productie

- Zet database en app in dezelfde regio/netwerkzone.
- Gebruik een connection pooler of beheerde Postgres-provider met PostGIS.
- Cache `/api/suggest` kort, maar sla ingevoerde adressen niet standaard permanent op.
- Voeg rate limiting toe voor publieke deployment.
- Toon altijd de actuele AERIUS-release.
- Draai bij iedere datasetupdate regressietests met vaste adressen/receptor-ID's.
- Gebruik een eigen/contractueel geschikte kaarttile-provider bij serieus publiek verkeer; de standaard OSM tile-server in deze starter is niet bedoeld voor zwaar productiegebruik.

## Juridische positionering

De tool geeft een geografische afstand tot relevante AERIUS-natuur. Dit is **geen** berekening van projectdepositie, geen ecologische beoordeling en geen oordeel over vergunningplicht. De tekst in de interface houdt dat onderscheid expliciet zichtbaar.

## Status / belangrijk

Dit is broncode, **geen gehoste website**. Er is nog geen echte landelijke AERIUS-import uitgevoerd of end-to-end test met echte PostGIS-data geslaagd in deze omgeving. De site kan pas afstanden geven nadat de officiële bestanden zijn geladen en gecontroleerd.

De adres-API gebruikt de officiële PDOK Locatieserver v3.1 (`/suggest` en `/free`) en leest `centroide_rd` als RD-coördinaten.

Controleer voor livegang of de geïmporteerde geometrie inderdaad 1-ha-hexagonen bevat, of de laagvelden exact overeenkomen en of de dichtstbijzijnde afstand met onafhankelijke GIS-steekproeven klopt. Zet geen demo-uitkomsten neer als echte metingen.
