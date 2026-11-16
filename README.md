# 🏥 MediGuard AI — Assistant Médical Big Data & RAG Hybride

Plateforme intelligente d'aide à la décision clinique permettant aux professionnels de santé d'interroger simultanément des **protocoles médicaux massifs** et l'**historique anonymisé des dossiers patients** en temps réel.

---

## 1. Vision & Architecture Globale

Le système repose sur deux pipelines complémentaires :
1. **Pipeline Big Data & Streaming (Asynchrone)** : Ingestion, nettoyage et anonymisation de volumes massifs de comptes-rendus cliniques (PDF, JSON, HL7) en batch et en temps réel via **Kafka, Spark, Hive et Pig**, alimentant de manière incrémentale une base vectorielle.
2. **Pipeline RAG Industriel (Temps Réel / Synchrone)** : Interface conversationnelle haute fiabilité combinant recherche hybride (dense + BM25), réordonnancement par **Cross-Encoder**, et garde-fous stricts (zéro hallucination, zéro fuite PII).

```text
[Sources Batch: MIMIC-IV / HAS] ──> [Pig / Spark Batch] ──> [Hive Warehouse]
                                            │ (Anonymisation PII + Chunking)
[Flux Live: Kafka (4 topics)] ───> [Spark Streaming] ─────┴──> [Qdrant (Dense+BM25)]
                                                                       │
[Médecin (React/Angular)] <──(SSE)── [FastAPI + Guardrails] <── [Cross-Encoder Top-5]
```

---

## 2. Jeux de Données (Datasets)

* **Dossiers Patients & Notes Cliniques** : [MIMIC-IV-Note](https://physionet.org) (comptes-rendus d'hospitalisation, résultats biologiques et prescriptions).
* **Protocoles & Littérature Médicale** : Guides cliniques de la [HAS (Haute Autorité de Santé)](https://has-sante.fr) et corpus [PubMed Central (PMC)](https://nih.gov) au format PDF et Markdown.
* **Flux Temps Réel (Kafka Topics)** : `vitals-iot-stream` (constantes vitales), `lab-results-hl7` (analyses biologiques), `clinical-notes-live` (admissions urgences), `prescriptions-events` & `protocols-updates` (pharmacovigilance et mise à jour de guides à chaud).

---

## 3. Stack Technique par Couches

| Couche Système | Technologies & Outils | Rôle & Justification Technique |
| :--- | :--- | :--- |
| **Ingestion & Streaming** | **Apache Kafka** + **Flume / Sqoop** | Gestion des 4 flux cliniques temps réel et import des bases relationnelles hospitalières. |
| **Traitement Big Data** | **Apache Spark** (PySpark, Structured Streaming) + **Hive** + **Pig** | Anonymisation PII (masquage noms/NSS), semantic chunking, embeddings distribués et stockage structuré sous Hive. |
| **Base Vectorielle** | **Qdrant** | Stockage des vecteurs denses et épars avec filtrage par métadonnées à chaud (spécialité médicale, date de validité). |
| **Base Relationnelle** | **PostgreSQL 16** | Gestion des utilisateurs, historique des sessions et journaux d'audit de conformité. |
| **Orchestration RAG & IA** | **LlamaIndex** + **`BAAI/bge-m3`** + **`BAAI/bge-reranker-large`** | Pipeline RAG Hybride (Dense + BM25) couplé à un Cross-Encoder réordonnant le Top-20 vers le Top-5. |
| **Backend API** | **FastAPI** + **Pydantic v2** | Endpoints REST et Streaming **SSE** asynchrones avec garde-fous anti-hallucination et RBAC (JWT). |
| **Frontend UI** | **React (Next.js)** ou **Angular 17+** | Interface médecin réactive affichant les réponses en streaming, alertes temps réel et citations exactes. |
| **MLOps & Évaluation** | **Ragas** + **Langfuse** | Traçabilité LLM et validation continue (`Faithfulness >= 0.90`, `Context Recall@5 >= 0.85`, latence `< 800 ms`). |
| **DevOps & Infra** | **Docker Compose** + **GitHub Actions** | Orchestration locale reproductible et pipeline CI/CD (linters + tests unitaires). |

---

## 4. Langages de Programmation

* **Python (3.11+)** : PySpark, FastAPI, orchestration RAG (LlamaIndex) et tests Ragas.
* **TypeScript (5.x)** : Développement de l'interface web sous React / Next.js ou Angular.
* **SQL / HiveQL** : Requêtes analytiques sous Apache Hive et gestion transactionnelle sous PostgreSQL 16.
* **Apache Pig Latin** : Scripts ETL de nettoyage initial des logs bruts sur HDFS.
* **YAML & Bash** : Configuration Docker Compose, CI/CD GitHub Actions et scripts de déploiement.

---

## 5. Arborescence du Dépôt GitHub

```text
medical-rag-bigdata/
├── .github/workflows/          # Pipelines CI/CD (pytest, ruff linter)
├── bigdata-pipelines/          # Scripts Big Data distribués
│   ├── spark_batch_pii.py      # Anonymisation PII et chunking distribué
│   ├── spark_streaming.py      # Consommation Kafka et vectorisation incrémentale
│   ├── hive_schemas.hql        # Tables Hive pour dossiers patients structurés
│   └── pig_logs_etl.pig        # Nettoyage des logs bruts
├── backend/                    # API FastAPI & Moteur RAG
│   ├── app/
│   │   ├── api/                # Routes REST & Streaming SSE
│   │   ├── core/               # Config, sécurité, Guardrails PII
│   │   ├── rag/                # Retrieval hybride, Cross-Encoder, prompts
│   │   └── models/             # Schémas Pydantic v2 & SQLAlchemy (PostgreSQL)
│   └── tests/                  # Tests unitaires & évaluation Ragas
├── frontend/                   # Application Web React (Next.js) ou Angular
├── data/
│   └── raw/                    # Échantillons de protocoles PDF/JSON de test
├── docker-compose.yml          # Orchestration de l'environnement
├── fiche_cadrage_projet.md     # Document de cadrage métier (5 dimensions)
├── matrice_moscow_backlog.md   # Backlog Agile MoSCoW
└── README.md                   # Documentation complète du projet
```

---

## 6. Démarrage Rapide (Environnement Local)

```bash
# 1. Cloner le dépôt et configurer les variables d'environnement
cp .env.example .env

# 2. Démarrer la base vectorielle Qdrant et PostgreSQL (Séance 2)
docker compose up -d qdrant postgres
```

---

## 7. Équipe & Répartition des Responsabilités (Binôme)

* **Membre 1 — Lead Data, Big Data & Moteur RAG** : Pipelines PySpark (anonymisation PII), tables Hive, topics Kafka, Semantic Chunking, indexation hybride Qdrant (`bge-m3`) et Cross-Encoder.
* **Membre 2 — Lead Backend IA, Interface & MLOps** : API FastAPI SSE, Guardrails anti-hallucination, interface web React/Angular, orchestration Docker Compose, CI/CD et évaluation Ragas/Langfuse.
