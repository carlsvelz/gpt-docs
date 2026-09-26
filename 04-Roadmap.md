# MACHINE LEARNING ENGINEERING — 2026

> **From Data → Models → Production → AI Systems**
> Roadmap de Z2H Academy. Construido desde cero con visión 2026: ML Engineering, no "Data Science tradicional".


## 1. Investigación de mercado (2026)

Antes de diseñar el plan de estudios miramos qué pide el mundo real. No enseñamos
lo que ya no se contrata; tampoco inflamos el programa con temas que no se aplican
en producción.

> 📝 **vacante** — puesto de trabajo abierto en una empresa; leer muchas juntas
> dice qué skills pesan de verdad.

### Demanda confirmada (5 fuentes, 2026)

| Fuente | Dato | Enlace |
|---|---|---|
| 365 Data Science | 1.144 vacantes ML Engineer: Python + SQL + ML + DL + cloud + Docker/K8s + Data Engineering como núcleo | https://365datascience.com/career-advice/career-guides/machine-learning-engineer-skills/ |
| S&P Global | Brechas 2026: ML/AI development, software engineering, data management | https://www.spglobal.com/en/research-insights/special-reports/ai-impact-on-employment-2026 |
| PwC AI Jobs Barometer | Skills de AI crecen más rápido que el mercado general | https://www.pwc.com/gx/en/news-room/press-releases/2026/pwc-2026-ai-jobs-barometer.html |
| Coursera | MLOps = especialización aparte (CI/CD, monitoring) | https://www.coursera.org/resources/mlops-learning-roadmap |
| Apiva (ago/26) | MLE combina ML + PyTorch + deploy + sistemas | https://kitejobs.ai/reports/machine-learning-engineer |

> 📝 **K8s** — Kubernetes, sistema para coordinar muchos contenedores como si fueran uno.

### Stack que entra (justificado por la tabla)

- Lenguaje: Python 3.12.
- Datos: SQL + pandas/Polars.
- ML clásico: scikit-learn 1.9 + XGBoost.
- Deep learning: PyTorch 2.x (CPU + CUDA).
- LLM: Hugging Face Hub + transformers/peft.
- Serving: FastAPI + pydantic v2 + Docker.
- Observabilidad: prometheus_client + métricas de modelo.
- Compute GPU en tiers: Lightning / Modal / Paperspace / Kaggle; SageMaker descartado.

### Stack que se descarta (con motivo)

- Excel / Tableau como eje: el mercado de MLE ya no los pide para construir sistemas.
- R como lenguaje principal: los roles modernos viven en Python + SQL.
- 30 algoritmos memorizados: el aprendizaje profundo cambia cómo se elige modelo.
- Matemática académica profunda previa: retrasa sin retorno en empleabilidad.
- Notebooks como producto final: el mercado pide APIs y servicios.
- Kaggle sin producción: la señal de portfolio es el sistema, no el ranking.
- CNN/Transformer desde cero como objetivo: ya existen preentrenados.
- GANs / RL como bloque central: casos de uso reducidos en vacantes MLE 2026.

### Conclusión

Este roadmap entrena al estudiante para ir de **datos → modelo → sistema → producto AI**,
no de Python → estadística → algoritmos → Kaggle.

MLOps vive como roadmap hermano (se cursa aparte). AI Engineering también.

---

## Posicionamiento Z2H

```
 DATA ENGINEERING        →  ¿Cómo conseguimos, almacenamos, transformamos y servimos datos confiables?
 MACHINE LEARNING        →  ¿Cómo aprendemos patrones de esos datos?
 MACHINE LEARNING ENG.   →  ¿Cómo convertimos ese modelo en un sistema reproducible y productivo?
 AI ENGINEERING          →  ¿Cómo construimos aplicaciones inteligentes con modelos fundacionales, agentes y datos?
```

Este roadmap es el puente: entra con datos confiables y sales con sistemas de ML en producción.

## Para quién SÍ es

- Hiciste `data-engineering` (o ya sabes SQL + Python) y quieres dar el salto a modelos.
- Quieres construir **sistemas** de ML (servir, monitorear, reentrenar), no solo notebooks.
- Quieres un perfil 2026: Python + ML + deep + cloud + Docker + pipelines.

## Para quién NO es

- Buscas estadística académica profunda o demostraciones matemáticas: no es este curso.
- Quieres solo teoría de algoritmos sin tocar producción: sobra la mitad del programa.
- Ya entrenas y despliegas modelos con MLOps: ve directo a `ai-engineering` (sistemas AI).

## Prerrequisitos honestos

- Python intermedio + SQL (equivalente a `data-engineering` nivel-1/2).
- Terminal y git sin miedo.
- Cero ML previo: este roadmap lo enseña desde el fundamento.

## Mapa (levels + sections)

| Level | Nombre | Pregunta que responde |
|---|---|---|
| 0 | ML Foundations | ¿Cómo aprende una máquina? |
| 1 | Data for ML | ¿Cómo preparamos datos para ML? |
| 2 | Classical ML | ¿Cómo construimos modelos? |
| 3 | Model Evaluation | ¿Cómo sabemos si funcionan? |
| 4 | Deep Learning | ¿Cómo construimos Deep Learning? |
| 5 | Modern Deep Learning | ¿Cómo funcionan los modelos modernos? |
| 6 | Model Adaptation | ¿Cómo adaptamos modelos preentrenados? |
| 7 | ML Software Engineering | ¿Cómo convertimos ML en software? |
| 8 | Production ML Systems | ¿Cómo construimos sistemas ML en producción? |
| 9 | ML + AI Engineering | ¿Cómo conectamos ML con AI moderna? |
| 10 | Capstone End2End | ¿Cómo construimos un sistema ML End2End? |

> Labs por level: pendientes de definir con sus tiers (GUIADO con el profe).
> Cada lab declarará: compute / providers / entrypoint / validation.

## 3.5 Dataset Strategy (estándar transversal)

Un dataset principal longitudinal + secundarios específicos cuando un
concepto los requiera. Cada Level desarrolla una **capacidad**; el
dataset es el **contexto** donde esa capacidad se ejercita.

Reglas duras:

- Descarga por **CLI/API** únicamente. Prohibido "entra a Kaggle y haz
  clic en Download". La provisión exacta de cada dataset vive en
  `section-1` (Workspace, se redacta al cerrar el stack).
- Un dataset por familia de problemas. No se inventan datasets
  intermedios; cada uno entra por un lab que lo justifique.

| # | Dataset      | Tipo           | CPU | GPU  | Función                  |
|---|--------------|----------------|-----|------|--------------------------|
| 01| NYC Taxi     | tabular grande | ✅  | opt. | **Principal** longitudinal |
| 02| UCI / OpenML | tabular limpio | ✅  | ❌   | Concept Labs (algoritmos puros) |
| 03| CIFAR-10     | visión         | ✅  | ✅   | Deep Learning introvisión |
| 04| NLP          | texto          | ✅  | ✅   | Transformers / LLM (TBD) |
| 05| End2End      | student solve  | —   | —    | Capstone L10 (problema **fijado por Z2H**, no elección libre) |

Provisioning NYC Taxi: NYC TLC oficial, descarga por CLI/API; el comando
esperado queda en `section-1`. NLP queda **TBD** hasta definir dirección
de L5/L9 con el profe.

### Asignación por Level

| Level | Datasets principales  | Uso                                              |
|-------|-----------------------|--------------------------------------------------|
| 0     | —                     | intro + workspace; sin dataset todavía           |
| 1     | NYC Taxi              | data prep, leakage, imbalanced, pipeline         |
| 2     | UCI / OpenML          | familias de modelos (regres/clasif/clustering)   |
| 3     | NYC Taxi              | métricas y selección sobre el principal          |
| 4     | CIFAR-10 + Taxi       | DL tabular + visión en GPU                       |
| 5     | CIFAR-10 + NLP (TBD)  | Transformers / Foundation Models                 |
| 6     | NLP (TBD)             | PEFT, RAG, embeddings                            |
| 7     | NYC Taxi              | empaquetado y serving                            |
| 8     | NYC Taxi              | batch / online inference                         |
| 9     | NLP (TBD)             | LLM / RAG / AI-assisted                          |
| 10    | End2End (Z2H-fixed)   | capstone                                         |

### Level 0 — ML Foundations

Objetivo: construir la mentalidad de ML sin convertir el nivel en un curso
universitario de matemáticas.

> Convención transversal (igual en todo roadmap): `section-0` = Welcome
> (intro + presentación del roadmap) · `section-1` = Configuración del
> Workspace (genérica: VM local / GitHub Codespaces / plataformas web como
> Kaggle, Colab, Databricks, Snowflake o equivalentes, según los labs).
>
> `section-1` se redacta al FINAL del desarrollo del roadmap, con el stack
> cerrado. Hasta entonces el archivo existe como reserva sin contenido y NO
> genera validación de contenido.

- **section-0 — Bienvenido al Roadmap de Machine Learning Engineering**:
  bienvenida con el nombre en negrita · desde dónde hasta dónde lleva el
  plan · objetivo como sistemas (construir / entrenar / desplegar / operar)
  · AI vs ML vs Deep Learning · Data Scientist vs ML Engineer vs AI
  Engineer · ML Engineering Lifecycle · From Notebook to Production · el
  nuevo rol del ML Engineer · contexto del mercado (las señales de demanda
  coinciden en Python + SQL + ML + deep learning + cloud + Docker + Data
  Engineering; brechas en ML/AI dev, software engineering y data
  management; ciclo completo, no solo notebooks; LLMs junto al ML
  tradicional; perfil híbrido).
- **section-1 — Configuración del Workspace** *(EN CONSTRUCCIÓN INCREMENTAL —
  v1 cubre provisión base validada; datasets y resto al cerrar stack)*:
  provisión del workspace con el stack cerrado:
  Python for ML · NumPy · Jupyter · Virtual environments · Git · Project
  structure · VM local + Ubuntu / GitHub Codespaces / plataformas web
  (Kaggle, Colab, Databricks, Snowflake, u otras) según los labs.
- **section-2 — How Machines Learn**: Dataset · Features · Labels ·
  Parameters · Hyperparameters · Training · Validation · Testing ·
  Inference · Generalization.
- **section-3 — The Mathematics of ML**: Vectors · Matrices · Dot Product ·
  Probability · Distributions · Mean / Variance · Derivatives · Gradients ·
  Gradient Descent.

### Level 1 — Data for ML

Aprovecha el background de Data Engineering de Z2H (no lo repite). Pregunta:
¿cómo transformamos datos en información utilizable por un modelo?

- **Section 0 — ML Dataset**: Dataset design · Samples · Features · Targets ·
  Training / Validation / Test datasets.
- **Section 1 — Data Preparation**: Missing values · Duplicates · Outliers ·
  Data types · Normalization · Standardization · Encoding.
- **Section 2 — Feature Engineering**: Numerical / Categorical / Temporal
  features · Transformations · Selection · Extraction.
- **Section 3 — Data Leakage**: What is leakage? · Target leakage ·
  Train/test contamination · Temporal leakage · Detection.
- **Section 4 — Imbalanced Data**: Class imbalance · Sampling · Oversampling ·
  Undersampling · Class weights · SMOTE.
- **Section 5 — ML Dataset Pipeline**: Raw Data → Cleaning → Transformation
  → Features → Training Dataset.

### Level 2 — Classical ML

No son 30 algoritmos: son las familias de modelos que siguen siendo relevantes.

- **Section 0 — Baselines**: Baseline models · Naive predictions ·
  Why baselines matter.
- **Section 1 — Regression**: Linear / Polynomial Regression · Regularization ·
  Ridge · Lasso.
- **Section 2 — Classification**: Logistic Regression · Binary / Multiclass
  classification.
- **Section 3 — Decision Trees**: Decision Trees · Random Forest ·
  Feature importance · Ensemble learning.
- **Section 4 — Gradient Boosting**: Boosting · XGBoost · LightGBM · CatBoost.
- **Section 5 — Unsupervised Learning**: Clustering · K-Means · Hierarchical
  clustering · PCA.
- **Section 6 — Anomaly Detection**: Statistical detection · Isolation Forest ·
  Outlier detection.
- **Section 7 — Scikit-Learn**: Pipelines · Transformers · Estimators ·
  Cross-validation · Model selection.

### Level 3 — Model Evaluation

Nivel completo (no un apéndice): ¿cómo demostramos que el modelo realmente funciona?

- **Section 0 — Train / Validation / Test**: Holdout · Cross-validation ·
  Stratified / Time-series splitting.
- **Section 1 — Regression Metrics**: MAE · MSE · RMSE · R² · MAPE.
- **Section 2 — Classification Metrics**: Accuracy · Precision · Recall · F1 ·
  ROC-AUC · PR-AUC.
- **Section 3 — Confusion Matrix**: True/False Positives · True/False Negatives.
- **Section 4 — Model Selection**: Hyperparameter tuning · Grid / Random Search ·
  Bayesian optimization.
- **Section 5 — Error Analysis**: Error distribution · Segment analysis ·
  Failure cases · Model limitations.
- **Section 6 — Business Evaluation**: Model Metric → Business Metric →
  Business Impact (diferencia con el ML académico).

### Level 4 — Deep Learning

Aquí empieza PyTorch (framework principal; no TensorFlow).

- **Section 0 — Neural Networks**: Perceptron · Neurons · Layers · Activation
  functions · Forward propagation.
- **Section 1 — Training Neural Networks**: Loss functions · Backpropagation ·
  Gradients · Optimizers · Learning rate.
- **Section 2 — PyTorch**: Tensors · Dataset · DataLoader · nn.Module ·
  Training loops · Autograd.
- **Section 3 — Neural Network Optimization**: Batch size · Learning rate ·
  Weight initialization · Dropout · Batch normalization · Weight decay.
- **Section 4 — GPU Computing**: CPU vs GPU · CUDA · GPU memory · Training on
  GPU · Mixed precision.
- **Section 5 — Deep Learning Project**: Dataset → Training → Evaluation →
  Experiment → Model.

### Level 5 — Modern Deep Learning

Eje: Transformers, pretrained y foundation models. NO el viejo camino
CNN → RNN → LSTM → GAN como eje.

- **Section 0 — Representation Learning**: Feature representation · Embeddings ·
  Learned representations · Transfer learning.
- **Section 1 — Computer Vision**: CNN fundamentals · Image classification ·
  Transfer learning · Object detection.
- **Section 2 — NLP**: Tokenization · Embeddings · Sequence modeling · Attention.
- **Section 3 — Transformers**: Self / Multi-head attention · Positional encoding ·
  Encoder · Decoder.
- **Section 4 — Foundation Models**: Pretrained models · Fine-tuning ·
  Inference · Checkpoints.
- **Section 5 — Hugging Face**: Transformers · Datasets · Tokenizers · Model Hub.

### Level 6 — Model Adaptation

Separar **usar** un modelo de **adaptarlo**.

- **Section 0 — Transfer Learning**: Pretrained models · Feature extraction ·
  Fine-tuning.
- **Section 1 — Fine-Tuning**: Full fine-tuning · Dataset preparation ·
  Training configuration · Evaluation.
- **Section 2 — Parameter-Efficient Fine-Tuning**: LoRA · QLoRA · PEFT.
- **Section 3 — Embeddings**: Text / Image embeddings · Similarity ·
  Semantic search.
- **Section 4 — Retrieval**: Vector representations · Vector indexes ·
  Retrieval · Reranking.
- **Section 5 — Model Optimization**: Quantization · Distillation · Pruning ·
  ONNX · Inference optimization (compresión para adaptar; el serving optimizado
  vive en Level 8).

### Level 7 — ML Software Engineering

Lo que lo hace ML Engineering y no Data Science con PyTorch: ¿cómo convertimos
el trabajo de ML en software profesional?

- **Section 0 — ML Project Architecture**: Project structure · Configuration ·
  Modules · Packages · Reusable components.
- **Section 1 — Production Python**: Type hints · Testing · Logging ·
  Error handling · Configuration management.
- **Section 2 — ML Pipelines as Software**: Data / Training / Evaluation /
  Inference pipelines.
- **Section 3 — Model APIs**: REST · FastAPI · Request validation · Response
  schemas · Model endpoints.
- **Section 4 — Packaging Models**: Artifacts · Serialization · Dependency
  management · Reproducibility.
- **Section 5 — Containers**: Docker · ML containers · GPU containers ·
  Reproducible environments.

### Level 8 — Production ML Systems

Frontera con MLOps: aquí se **construye el sistema**; operar y automatizar su
ciclo de vida es del roadmap hermano.

- **Section 0 — Batch Inference**: Data → Model → Predictions → Storage.
- **Section 1 — Online Inference**: Application → API → Model → Prediction.
- **Section 2 — Model Serving**: Model servers · Latency · Throughput · Batching.
- **Section 3 — Inference Optimization**: CPU / GPU inference · Quantization ·
  Batching · Caching.
- **Section 4 — Distributed ML**: Data / Model parallelism · Distributed
  training · Distributed inference.
- **Section 5 — ML System Design**: Architecture · Scalability · Reliability ·
  Latency · Cost · Failure modes.

### Level 9 — ML + AI Engineering

Actualización 2026: sin convertir ML Engineering en AI Engineering. Un ML
Engineer moderno entiende cómo sus modelos participan en sistemas de AI.

- **Section 0 — LLM Engineering**: LLM architecture · Tokenization · Context ·
  Inference · Model selection.
- **Section 1 — LLM Fine-Tuning**: Instruction datasets · SFT · LoRA · QLoRA ·
  Evaluation.
- **Section 2 — RAG Systems**: Embeddings · Retrieval · Vector search ·
  Reranking · Context construction.
- **Section 3 — AI Evaluation**: LLM evaluation · Grounding · Hallucination ·
  Evaluation pipelines.
- **Section 4 — AI Inference**: Open-source models · vLLM · GPU inference ·
  Quantization · Throughput · Latency.
- **Section 5 — AI-assisted ML Engineering**: AI-assisted coding /
  experimentation / debugging · Coding agents · Agentic workflows (conexión
  directa con Z2H Tutor).

### Level 10 — Capstone: End-to-End ML Engineering

Un proyecto grande (no varios pequeños): atravesar todo el ciclo y terminar
pudiendo decir **"I built and deployed a Machine Learning system"** (no "I
trained a model in a notebook").

Fases: Dataset → Data Preparation → Feature Engineering → Baseline → Model
Training → Evaluation → Deep Learning → Model Selection → Model Package →
Model Server → Inference → AI Integration.

> El Capstone es proyecto, no lecciones: no genera archivos `section-*.md`.

 ¿Cómo uso AI para construir ML? |

## Qué NO es el eje (2026)

- Excel / Tableau / R como lenguaje principal.
- 30 algoritmos clásicos memorizados.
- Matemática académica profunda antes de tocar ML.
- Notebooks como producto final.
- Kaggle sin producción.
- Construir CNN/Transformers desde cero como objetivo.
- GANs o RL como bloque central.

## Reglas

- Herramientas de un nivel NO se usan en niveles anteriores.
- Cada nivel cierra con ejercicios; soluciones en `apendices/E-soluciones.md`.
- Convenciones: `level-N/`, secciones `section-X.md`, español neutro sin voseo.
