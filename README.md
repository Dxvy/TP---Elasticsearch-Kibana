# TP---Elasticsearch-Kibana

### Partie 1: Installation et Configuration du Cluster Elasticsearch

1. **Installation de Elasticsearch via Docker :**
   ```
   docker pull docker.elastic.co/elasticsearch/elasticsearch:7.15.1
   ```

2. **Création du Cluster Elasticsearch :**
    - Créer un fichier `docker-compose.yml` avec la configuration pour plusieurs instances Elasticsearch en mode
      cluster.

3. **Démarrage du Cluster :**
   ```
   docker-compose up
   ```

### Partie 2: Premiers Pas avec le Cluster Elasticsearch

1. **Création de l'index test01 et insertion du document :**
   ```
   curl -X PUT "localhost:9200/test01" -H 'Content-Type: application/json' -d'
   {
     "titre": "Premier document",
     "description": "C'est la première fois que je crée un document dans ES.",
     "quantité": 12,
     "date_creation": "10-05-2024"
   }'
   ```

2. **Affichage du document et vérification du mapping :**
   ```
   curl -X GET "localhost:9200/test01/_doc/1"
   curl -X GET "localhost:9200/test01/_mapping"
   ```

3. **Création de l'index test02 et mapping adapté :**
   ```
   curl -X PUT "localhost:9200/test02" -H 'Content-Type: application/json' -d'
   {
     "mappings": {
       "properties": {
         "titre": { "type": "text" },
         "description": { "type": "text" },
         "quantité": { "type": "integer" },
         "date_creation": { "type": "date", "format": "dd-MM-yyyy" }
       }
     }
   }'
   ```

4. **Affichage du traitement de la description par l'analyzer :**
   ```
   curl -X GET "localhost:9200/test02/_analyze" -H 'Content-Type: application/json' -d'
   {
     "text": "C'est la première fois que je crée un document dans ES."
   }'
   ```

5. **Création de l'index test03 avec mapping spécifique :**
   ```
   curl -X PUT "localhost:9200/test03" -H 'Content-Type: application/json' -d'
   {
     "mappings": {
       "properties": {
         "description": {
           "type": "text",
           "fields": {
             "raw": {
               "type": "keyword"
             }
           }
         }
       }
     }
   }'
   ```

6. **Import multiple de 4 documents en une seule requête :**
   ```
   curl -X POST "localhost:9200/test03/_bulk" -H 'Content-Type: application/json' -d'
   { "index": { "_index": "test03" }}
   { "titre": "Document 1", "description": "Premier document", "quantité": 5, "date_creation": "15-05-2024" }
   { "index": { "_index": "test03" }}
   { "titre": "Document 2", "description": "Deuxième document", "quantité": 8, "date_creation": "20-05-2024" }
   { "index": { "_index": "test03" }}
   { "titre": "Document 3", "description": "Troisième document", "quantité": 3, "date_creation": "25-05-2024" }
   { "index": { "_index": "test03" }}
   { "titre": "Document 4", "description": "Quatrième document", "quantité": 10, "date_creation": "30-05-2024" }
   ' 
   ```

### Partie 3: Intégration de Kibana

### Intégration des données factices et requêtes avec Kibana :

1. **Peupler le cluster avec les données factices :**

```
curl -XPOST "http://localhost:9200/votre_index/votre_type" -H 'Content-Type: application/json' -d '{
  "num_vol": "ABC123",
  "compagnie": "AirFrance",
  "prix": 400,
  "statut": "confirmé",
  "pluie_arrivée": false,
  "pluie_départ": true,
  "départ": "Allemagne",
  "destination": "France",
  "date": "2024-04-15"
}'
```

2. **Lancement de Kibana :**
    - Démarrez un conteneur Kibana en utilisant la commande suivante :
      ```
      docker run --name kibana -d --link elasticsearch1:elasticsearch -p 5601:5601 docker.elastic.co/kibana/kibana:7.15.1
      ```

3. **Accès à l'interface de Kibana :**
    - Accédez à l'interface de Kibana via votre navigateur en ouvrant `http://localhost:5601`.

4. **Réalisations des requêtes avec Kibana :**

   - **Lister les vols dont le prix moyen est entre 300€ et 450€ :**
      ```bash
      curl -XGET 'http://localhost:9200/your_index/_search?q=prix:[300 TO 450]'
      ```

   - **Lister les vols annulés :**
      ```bash
      curl -XGET 'http://localhost:9200/your_index/_search?q=statut:annulé'
      ```

   - **Lister les vols avec de la pluie à l'arrivée ou au départ, priorisant l'arrivée :**
      ```bash
      curl -XGET 'http://localhost:9200/your_index/_search?q=pluie_arrivée:true OR pluie_départ:true&sort=pluie_arrivée:desc'
      ```

   - **Lister les vols partant d'Allemagne et à destination de la France :**
      ```bash
      curl -XGET 'http://localhost:9200/your_index/_search?q=départ:Allemagne AND destination:France'
      ```

   - **Lister les vols ayant eu lieu entre le 1er avril 2024 et le 20 mai 2024 :**
      ```bash
      curl -XGET 'http://localhost:9200/your_index/_search?q=date:[2024-04-01 TO 2024-05-20]'
      ```

