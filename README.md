# 🏥 MediGuard AI — Plateforme Big Data & RAG Clinique

Assistant intelligent d'aide à la décision médicale croisant des protocoles cliniques massifs et des dossiers patients anonymisés via une architecture Big Data (**Spark, Hive, Kafka**) et un moteur **RAG Hybride** (**Qdrant, Cross-Encoder, FastAPI**).

## 🏗️ Architecture Technique
* **Big Data & Streaming** : Apache Kafka (4 topics temps réel), Apache Spark (PySpark / Structured Streaming), Apache Hive, Pig.
* **Moteur RAG & IA** : Qdrant (Dense + Sparse BM25), Embeddings `BAAI/bge-m3`, Reranker `BAAI/bge-reranker-large`, LlamaIndex.
* **Backend & Frontend** : FastAPI (Python 3.11, Pydantic v2, SSE) + React / Next.js (TypeScript).
* **Qualité & DevOps** : Ragas, Langfuse, Docker Compose, GitHub Actions.

## 🚀 Démarrage Rapide (Séance 2)
```bash
cp .env.example .env
docker compose up -d qdrant postgres
```

## 👥 Équipe & Répartition des Responsabilités (Binôme)
* **Membre 1 — Lead Data, Big Data & Moteur RAG** : Pipelines PySpark (anonymisation PII), tables Hive, topics Kafka, Semantic Chunking, indexation hybride Qdrant (`bge-m3`) et Cross-Encoder.
* **Membre 2 — Lead Backend IA, Interface & MLOps** : API FastAPI SSE, Guardrails anti-hallucination, interface web React/Angular, orchestration Docker Compose, CI/CD et évaluation Ragas/Langfuse.
