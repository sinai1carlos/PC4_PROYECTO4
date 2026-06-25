# Proyecto 4: Arquitectura RAG básica — Detección de contradicciones normativas

**Curso:** CC0C2 — Procesamiento del Lenguaje Natural — Práctica Calificada 4
**Cuaderno base:** `Cuaderno22-CC0C2.ipynb`
**Autor:** Carlos Sinai Unda Miguel

---

## Objetivo

Reconstruir la línea base `rag_tool()` del Cuaderno22 (retrieval léxico mock sobre un diccionario de 9 entradas) y compararla contra un pipeline RAG real con embeddings densos, vector store, clasificación NLI y generación fundamentada, aplicado a la detección de contradicciones normativas en el Código Civil Peruano.

Este proyecto es una versión reducida y reproducible de mi tesis: *"Detección Automática de Contradicciones Normativas en Textos Jurídicos Peruanos Mediante RAG e Inferencia de Lenguaje Natural"*.

---

## Resumen de la línea base

`rag_tool()` del Cuaderno22 (Sección 11) realiza retrieval por **coincidencia léxica de palabras clave** sobre un diccionario `KNOWLEDGE_BASE` de 9 entradas. No usa embeddings ni vector store. Funciona para demostrar el concepto de RAG pero falla ante sinónimos o paráfrasis (demostrado en la Sección 2 del notebook).

---

## Modificación realizada

Pipeline RAG real de 4 componentes:

| Componente | Tecnología | Reemplaza a |
|---|---|---|
| Embeddings | `BAAI/bge-m3` (1024 dims) | Coincidencia léxica |
| Vector store | Qdrant (modo local) | Diccionario Python |
| Clasificación de relación | `cross-encoder/nli-deberta-v3-base` | (no existía en el mock) |
| Generación fundamentada | gemini-2.5-flash-lite | (no existía en el mock) |

**Ejercicio A (común):** cambio de `top_k` (2, 5, 10) y análisis de precisión/cobertura.
**Ejercicio B (específico):** agregar reranking por densidad de términos jurídicos antes del clasificador NLI.

---

## Corpus utilizado

- **Fuente:** Código Civil Peruano (Decreto Legislativo Nº 295), PDF oficial de 453 páginas.
- **Extracción:** `pdfplumber` + segmentación por artículo con expresiones regulares.
- **Alcance:** Libro I — Personas Naturales, Artículos 1 a 100 (100 artículos indexados).
- **Hallazgo clave:** 14 artículos tienen múltiples versiones históricas dentro del documento (texto original + texto modificado por reforma posterior), usadas como base del gold dataset.

## Gold dataset

18 pares anotados manualmente (criterio de antinomia normativa, Bobbio 1994):
- 7 `CONTRADICTION` (prescripciones jurídicas incompatibles)
- 5 `ENTAILMENT` (mismo contenido normativo, cambio de forma/terminología)
- 6 `NEUTRAL` (artículos no relacionados o con supuestos de hecho distintos)

---

## Cómo ejecutar el notebook

```bash
pip install sentence-transformers qdrant-client transformers google-generativeai torch scikit-learn pandas matplotlib -q
```

1. Configurar variable de entorno `GOOGLE_API_KEY` con tu API key de [Google AI Studio](https://aistudio.google.com/apikey).
2. Asegurar que `corpus_principal.json` y `gold_dataset_final.json` estén en el mismo directorio que el notebook.
3. Ejecutar todas las celdas en orden (`Run All`).

**Semilla:** `42` (reproducible)
**Tiempo estimado en CPU:** ~10-15 minutos (la mayor parte es generar embeddings + clasificación NLI)

---

## Principales resultados

- El retrieval léxico del Cuaderno22 falla ante paráfrasis; el retrieval semántico las resuelve correctamente.
- Accuracy del clasificador NLI sobre el gold dataset jurídico: ver Sección 5.2 del notebook tras ejecución.
- El reranking (Ejercicio B) cambia efectivamente el conjunto de candidatos evaluados por el NLI, confirmando que no es un cambio cosmético.

---

## Limitaciones

1. Corpus reducido (100 de >2,300 artículos extraídos).
2. Gold dataset pequeño (18 pares, un solo anotador, sin kappa de Cohen).
3. NLI sin fine-tuning específico al dominio jurídico peruano.
4. Solo detecta contradicciones explícitas (no implícitas/contextuales).
5. Extracción de PDF con ruido residual de notas de concordancia.

---

## Qué se muestra en el video

1. Presentación del proyecto y cuaderno base (Cuaderno22)
2. Explicación de la línea base `rag_tool()` (retrieval léxico) y su limitación
3. **Codificación en vivo — Ejercicio A:** cambio de `top_k`, predicción e interpretación
4. **Codificación en vivo — Ejercicio B:** implementación de reranking jurídico, predicción e interpretación
5. Explicación de embeddings (shape, normalización), scores de similitud coseno y matriz de confusión NLI
6. Comparación línea base vs sistema propuesto (tabla Sección 10)
7. Caso de estudio real (Art. 42, capacidad de ejercicio) con justificación generada por Gemini
8. Respuesta a 5 preguntas avanzadas + 5 transversales
9. Limitaciones y cierre técnico
10. Puente al curso (conexión con Cuaderno23 y mi tesis)

---

## Declaración de autoría y uso de IA

```
Declaro que comprendo el código, los resultados y las explicaciones entregadas en esta práctica.
Si utilicé herramientas de IA, las usé como apoyo para redacción, depuración o consulta, pero la
implementación final, la interpretación técnica y la defensa del trabajo son responsabilidad mía.
```

Usé IA (Claude) para generar una primera versión de las funciones de retrieval semántico, reranking y el pipeline de extracción/limpieza del PDF, que revisé, ejecuté y expliqué celda por celda. La anotación del gold dataset, la selección del corpus y la interpretación de resultados son responsabilidad mía.
