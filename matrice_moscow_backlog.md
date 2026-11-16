# Backlog Agile MoSCoW — Projet MediGuard AI

## MUST HAVE (Indispensables pour le jalon final S7)
* **US-01 (Ingestion Big Data & Anonymisation PII)** : *En tant que Data Engineer, je veux traiter les comptes-rendus patients et protocoles PDF via PySpark et Hive en masquant les PII avant indexation, afin de garantir la conformité RGPD/secret médical.*
  * **Critères d'acceptation** : 100% des noms et identifiants patients masqués (`[REDACTED]`) sur un échantillon test de 1 000 documents ; tables structurées créées sous Hive.
* **US-02 (Retrieval Hybride & Reranking)** : *En tant que médecin, je veux que le moteur combine recherche vectorielle dense et recherche lexicale BM25 avec un Cross-Encoder, afin de trouver les protocoles pertinents même avec des abréviations cliniques.*
  * **Critères d'acceptation** : `Context Recall@5` $\geq 0.85$ et `NDCG@5` $\geq 0.80$ mesurés sur un jeu de test de 50 questions médicales.
* **US-03 (Génération Sourcée & Garde-fous Anti-Hallucination)** : *En tant que médecin urgentiste, je veux obtenir une synthèse clinique citant exactement les pages des protocoles sources et refusant de répondre hors contexte, afin de sécuriser ma décision.*
  * **Critères d'acceptation** : Score Ragas `Faithfulness` $\geq 0.90$ ; présence systématique de citations cliquables ; rejet 100% des questions hors domaine médical.
* **US-04 (API FastAPI Streaming & Interface Web)** : *En tant qu'utilisateur, je veux poser ma question sur une interface React/Angular connectée à FastAPI en streaming SSE, afin d'obtenir une réponse fluide en moins de 800 ms.*
  * **Critères d'acceptation** : Latence *First-Token* $< 800\text{ ms}$ et affichage des extraits sources dans l'interface.

## SHOULD HAVE (Importants mais non bloquants)
* **US-05 (Ingestion Temps Réel Kafka + Spark Streaming)** : *Ingérer en continu les 4 flux cliniques (constantes IoT, bilans labo, notes live, prescriptions/protocoles) via Kafka et Spark Structured Streaming pour alerter le médecin en temps réel.*
* **US-06 (Observabilité MLOps avec Langfuse)** : *Tracer toutes les requêtes, scores de reranking et coûts de tokens dans Langfuse.*

## COULD HAVE (Bonus à forte valeur ajoutée)
* **US-07 (GraphRAG Interactions Médicamenteuses)** : *Coupler Qdrant à Neo4j pour modéliser les graphes d'interactions entre molécules et pathologies chroniques.*

## WON'T HAVE (Hors périmètre S7)
* **US-08 (Analyse d'imagerie médicale DICOM / Radiologie)** : *Le projet se limite aux données textuelles et tabulaires.*
* **US-09 (Déploiement Cloud multi-cluster Kubernetes)** : *L'exécution se fera via Docker Compose en environnement local/serveur.*
