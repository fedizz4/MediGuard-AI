# Fiche de Cadrage — Assistant Médical Big Data & RAG Hybride (MediGuard AI)

## 1. Description du Problème Métier
* **Persona Utilisateur** : Médecins urgentistes, internes et praticiens hospitaliers en service de soins critiques.
* **Point de Douleur (Pain Point)** : Lors d'une prise en charge urgente, le médecin doit croiser en quelques minutes des milliers de pages de protocoles cliniques (HAS, OMS) avec l'historique massif et les données en direct du patient (comptes-rendus, constantes vitales, bilans biologiques, ordonnances).
* **Impossibilité d'une recherche lexicale classique** : Une recherche par mots-clés (SQL `LIKE` ou BM25 pur) échoue face aux abréviations médicales (ex. *HTA*, *BPCO*), aux synonymes pharmacologiques (DCI vs nom commercial) et à l'absence de compréhension du contexte clinique global du patient.
* **Plus-value de l'IA Générative + Big Data** : Apache Spark et Hive nettoient, structurent et anonymisent des volumes massifs de données hospitalières en batch et en streaming, tandis que le moteur RAG hybride synthétise les protocoles applicables avec des citations exactes et zéro hallucination.

## 2. Source & Volumétrie des Données
* **Sources de Données Statiques & Batch** :
  * *Dossiers Patients & Notes Cliniques* : Dataset ouvert **MIMIC-IV-Note** (`https://physionet.org`) entreposé sous **Apache Hive**.
  * *Protocoles Médicaux* : Guides cliniques **HAS** (`https://has-sante.fr`) et corpus **PubMed Central** (`https://nih.gov`) au format PDF et Markdown.
* **Flux Temps Réel (Apache Kafka + Spark Structured Streaming)** :
  1. **Topic `vitals-iot-stream`** : Constantes vitales continues (fréquence cardiaque, tension artérielle, SpO2) issues des moniteurs de surveillance pour déclencher des alertes d'urgence.
  2. **Topic `lab-results-hl7`** : Résultats d'analyses biologiques validés par les laboratoires (format HL7/JSON) pour détecter les seuils critiques.
  3. **Topic `clinical-notes-live`** : Observations et symptômes saisis en direct par les soignants lors de l'admission aux urgences.
  4. **Topic `prescriptions-events` & `protocols-updates`** : Nouvelles ordonnances (contrôle des interactions médicamenteuses) et nouveaux guides cliniques HAS/OMS (vectorisation incrémentale à chaud).
* **Volumétrie Initiale (Test / POC)** : 500 protocoles PDF (~15 000 pages) + 50 000 comptes-rendus patients anonymisés (~2 Go) + simulateur Kafka à 50 événements/seconde.
* **Volumétrie Cible (Production)** : 10 000 protocoles + 5 millions de dossiers patients (~500 Go), justifiant le traitement distribué sous **Spark** et l'entreposage sous **Hive**.

## 3. Stratégie d'Ingestion & Vectorisation
* **Prétraitement Batch & Streaming (Spark & Pig)** : Scripts **PySpark** (batch et Structured Streaming) pour l'extraction de texte PDF, le nettoyage initial des logs bruts via **Apache Pig**, et le stockage des métadonnées structurées dans **Apache Hive**.
* **Vectorisation Incrémentale** : Tout nouveau protocole publié sur `protocols-updates` est découpé et vectorisé à la volée par Spark Streaming pour mettre à jour **Qdrant** sans interruption de service.
* **Découpage (Chunking)** : **Semantic Chunking** couplé à une approche *Parent-Document* (chunks enfants de 512 tokens avec overlap de 15% rattachés au paragraphe parent complet) pour préserver la cohérence des posologies médicales.
* **Modèle d'Embedding** : `BAAI/bge-m3` (multilingue français/anglais, supportant nativement les vecteurs denses et épars pour le hybride).
* **Base Vectorielle & Reranking** : **Qdrant** (choisi pour son filtrage par métadonnées à chaud ultra-rapide par spécialité médicale et date de validité) couplé à un **Cross-Encoder** (`BAAI/bge-reranker-large`) réordonnant le Top-20 vers le Top-5.

## 4. Métriques de Qualité & SLA
* **KPIs de Retrieval** :
  * `Context Recall@5` $\geq 0.85$
  * `NDCG@5` $\geq 0.80$
* **KPIs de Génération (Framework Ragas)** :
  * `Faithfulness` (Fidélité stricte aux protocoles sources) $\geq 0.90$
  * `Answer Relevance` $\geq 0.85$
* **SLA Technique** :
  * Latence *First-Token* (Streaming SSE) $< 800\text{ ms}$
  * Latence pipeline streaming Kafka $\rightarrow$ Spark $\rightarrow$ Alerte RAG $< 3\text{ secondes}$

## 5. Garde-fous & Sécurité
* **Filtrage PII (Données Personnelles)** : Pipeline **Spark NLP / Presidio** exécuté *avant* toute écriture dans Hive ou Qdrant pour masquer automatiquement les noms, numéros de sécurité sociale (NSS), dates de naissance exactes et adresses.
* **Détection des Hallucinations** : Prompt système strict imposant le refus de réponse si le score de confiance du Reranker est $< 0.65$, couplé à une vérification de citations obligatoires `[Source, Page]`.
* **Gestion des Accès (RBAC)** : Filtrage par métadonnées dans Qdrant et authentification JWT sur **FastAPI** selon le service hospitalier du praticien (ex. Urgentiste vs Cardiologue).
