# Dual-RAG Code Debugger

> Inteligencia de código sobre un **grafo de conocimiento** multi-lenguaje: indexas un repo, preguntas en lenguaje natural y obtienes diagnósticos **anclados a símbolos reales** — no alucinaciones sueltas.

<!-- ============================================================
     VIDEO DE PRESENTACIÓN
     Pega aquí el enlace o el embed de YouTube cuando lo tengas.
     Ejemplo embed:

     [![Dual-RAG en 3 minutos](https://img.youtube.com/vi/ID_DEL_VIDEO/maxresdefault.jpg)](https://www.youtube.com/watch?v=ID_DEL_VIDEO)

     O solo el link:
     **Video:** https://www.youtube.com/watch?v=ID_DEL_VIDEO
     ============================================================ -->

**Video de presentación:** _[ ← pega aquí el enlace de YouTube ]_

[![Estado](https://img.shields.io/badge/estado-activo-brightgreen)](#)
[![Python](https://img.shields.io/badge/python-3.11%2B-blue)](#requisitos)
[![Benchmark](https://img.shields.io/badge/benchmark-docs%2Fbenchmark.md-orange)](docs/benchmark.md)

**Análisis frío de capacidades (lenguajes, scores, límites):** → **[docs/benchmark.md](docs/benchmark.md)**

---

## Qué problema resuelve

Cuando un bug cruza varios archivos, lenguajes o capas (API, HTML, deps, deploy), el chat genérico **inventa** o se pierde. Dual-RAG:

1. **Indexa** el proyecto a un grafo (símbolos, imports, calls, herencia, assets, deps).
2. **Ancla** cada respuesta a evidencia del grafo del **proyecto activo**.
3. **Declara** el dueño del fallo: código · entorno · grafo incompleto · incierto.
4. Opcionalmente **genera tests**, los ejecuta y corrige en bucle (fuerte en Python; parcial en JS/Go).

No es un debugger con breakpoints. Es un **orquestador de evidencia estructural** para entender y diagnosticar.

---

## Capturas de pantalla

> Coloca las imágenes en `docs/images/` con los nombres indicados.  
> **Qué fotografiar:** lista al final de este README → [Guía de capturas](#guía-de-capturas-qué-fotografiar).

### Interfaz principal (mapa molecular + chat)

<!-- FOTO 1: pantalla completa con chat a la izquierda y esfera "Listo" / grafo a la derecha -->
![Interfaz principal Dual-RAG](docs/images/01-ui-overview.png)

_Sistema listo, sidebar de proyectos, motor LLM y mapa molecular._

### Indexación de un proyecto

<!-- FOTO 2: tras "Cargar / indexar" — status System Ready + nodos reales en el mapa -->
![Proyecto indexado](docs/images/02-project-indexed.png)

_Proyecto activo, nodos de archivos (esferas) y enlaces entre módulos._

### Diagnóstico anclado al grafo

<!-- FOTO 3: respuesta del chat con resumen, confianza, insight y relaciones -->
![Diagnóstico con evidencia](docs/images/03-diagnosis-evidence.png)

_Resumen en español, owner del fallo, relaciones CALLS/CONTAINS/INHERITS, etc._

### Generación de pruebas

<!-- FOTO 4: menú + / test module|e2e y resultado passed o skipped_env -->
![Test generation](docs/images/04-testgen-result.png)

_Estado honesto: `passed` vs `skipped_env` (fallo de entorno, no del debugger)._

### Capas de negocio (L1/L2)

<!-- FOTO 5: pregunta de "flujo de negocio" con pipeline y contenedores lógicos -->
![Flujo de negocio L1/L2](docs/images/05-business-layers.png)

_Objetivo del proyecto, pipeline y roles de carpetas — no solo CALLS locales._

---

## Cómo funciona (arquitectura en 30 segundos)

```
  Repo en disco
       │
       ▼
  Indexer ──► Grafo (L3) + capas L1/L2 (negocio) + L4 (deps/versiones)
       │
       ▼
  Pregunta ──► Bootstrap local (0 LLM) ──► Evidencia anclada
       │
       ▼
  LLM (Groq / Gemini / …) solo con el pack de evidencia
       │
       ▼
  Respuesta: resumen · owner · relaciones · confianza
       │
       └── (opcional) Testgen → escribir → pytest/node/go → fix loop
```

| Capa | Qué guarda |
|------|------------|
| **L1** | Objetivo general del proyecto, entrypoints, dominios |
| **L2** | Roles por módulo/carpeta (orchestration, llm, deploy…) |
| **L3** | Símbolos y relaciones finas (CALLS, CONTAINS, INHERITS…) |
| **L4** | Dependencias y versiones (requirements, locks, runtime) |

Detalle de madurez por lenguaje y scores: **[benchmark.md](docs/benchmark.md)**.

---

## Requisitos

- **Python 3.11+** (probado en 3.12)
- Sistema: Windows 11 / Linux / macOS
- API keys **opcionales** al arrancar (`.env`): el sistema no exige todas

```env
# .env (ejemplo — copia desde .env.example)
GROQ_API_KEY=
GEMINI_API_KEY=
# CEREBRAS_API_KEY=
# CLOUDFLARE_API_TOKEN=
# CLOUDFLARE_ACCOUNT_ID=
```

---

## Instalación rápida

```bash
git clone https://github.com/StevAL2000/Ia-Debugger.git
cd Ia-Debugger

# Linux / macOS
chmod +x start.sh
./start.sh

# Windows (PowerShell)
.\start.ps1
# o: start.bat
```

Los scripts crean el venv, instalan `requirements.txt` si hace falta, levantan la API y pueden abrir el navegador.

**Manual:**

```bash
python -m venv venv
# Windows: venv\Scripts\activate
# Linux/macOS: source venv/bin/activate
pip install -r requirements.txt
python main.py --project /ruta/a/tu/codigo
```

API por defecto: **http://127.0.0.1:5000/**

---

## Uso paso a paso

### 1. Arrancar

Con `start.sh` / `start.ps1` o:

```bash
python main.py --project "C:\ruta\a\tu\repo"
```

### 2. Elegir o indexar proyecto (UI)

1. Abre http://127.0.0.1:5000/
2. En el menú izquierdo:
   - **Proyectos guardados** → selecciona un grafo ya indexado → **Usar**
   - O pega una **Nueva ruta** → **Cargar / indexar**
3. Espera a **System Ready** (verde)

<!-- FOTO 6: detalle del sidebar de proyectos (Usar / grafo / todo + ruta) -->
![Sidebar proyectos](docs/images/06-sidebar-projects.png)

### 3. Preguntar

Escribe en el chat (Enter envía · Shift+Enter nueva línea) y pulsa **Analizar**.

Ejemplos útiles:

- `¿Dónde está definida la clase X y quién la usa?`
- `analiza el flujo de negocio del proyecto`
- `¿formulario.js es llamado por otros .html dentro de producto_0003?`
- `main.py, ¿usa realmente la carpeta producto_0003?`

### 4. Explorar el mapa molecular

- **Arrastrar** = rotar  
- **Scroll** = zoom  
- **Clic en átomo** = centrar  
- Panel **Nodos & Enlaces**: archivos (esferas) y relaciones  

<!-- FOTO 7: mapa con varias esferas y enlaces iluminados tras una consulta -->
![Mapa molecular con evidencia](docs/images/07-molecular-map-evidence.png)

### 5. Generar pruebas (botón +)

Tres modos:

| Modo | Qué hace |
|------|----------|
| **E2E live** | Prueba de ejecución real (sin mocks de dominio) |
| **E2E mock** | E2E con mocks controlados |
| **Módulo/carpeta** | Test del target que elijas en la lista |

El sistema usa el grafo, escribe el test, intenta ejecutarlo y puede corregir (máx. intentos).  
Si falla la **infra** (DB caída, paquete ausente), el estado debe ser **`skipped_env`**, no un falso “passed”.

### 6. Cambiar motor LLM

En **Engine Mode** elige Groq, Gemini, etc. Las keys se leen del `.env`; no hace falta reiniciar si ya estaban cargadas al arranque (según configuración).

### 7. Reindexar sin empezar de cero

Si el banner indica grafo desactualizado:

- Preferir **actualización incremental** (solo archivos dirty)
- Full reindex solo si hubo corrupción o primer index

---

## Lenguajes soportados (resumen)

| Nivel | Lenguajes |
|-------|-----------|
| **Fuerte (grafo + diagnóstico)** | Python, C/C++, Rust, Solidity, HTML/CSS |
| **Bueno** | JavaScript/TypeScript, Go, Shell |
| **Básico / parcial** | YAML, Dockerfile, PHP, Java, C#, SQL |

Tabla completa y scores: **[docs/benchmark.md](docs/benchmark.md)**.

---

## Estructura del repositorio (orientativa)

```
dual_rag_debugger/          # o raíz del repo según tu layout
├── main.py                 # entrada CLI / API
├── analysis/               # parsers multi-lang (lang_cpp, lang_rust, lang_solidity, …)
├── ingestion/              # indexación e incremental
├── knowledge/              # grafo + capas L1–L4
├── reasoning/              # orquestador, testgen, validación
├── interface/              # Flask + frontend (mapa 3D, chat)
├── llm/                    # gateway Groq / Gemini / …
├── config/                 # settings, constantes
├── docs/
│   ├── benchmark.md        # ← análisis frío
│   └── images/             # capturas del README
├── requirements.txt
├── start.sh / start.ps1 / start.bat
└── .env.example
```

---

## Configuración mínima

1. Copia `.env.example` → `.env`
2. Pon al menos una key (p. ej. `GROQ_API_KEY`) para chat con modelo en la nube
3. Sin keys: puedes indexar y explorar el grafo; el razonamiento LLM quedará limitado

---

## Buenas prácticas

- **Siempre** confirma el proyecto activo antes de diagnosticar otro repo.
- Preguntas con **nombre de archivo/símbolo** anclan mejor que frases vagas.
- Si la confianza es baja o dice “evidencia incompleta”: reindexa o nombra el path exacto.
- No interpretes `skipped_env` como bug del debugger: revisa Postgres/Redis/deps del **proyecto bajo test**.

---

## Roadmap (honesto)

Ver también [STRATEGY_GOD_DEBUGGER.md](STRATEGY_GOD_DEBUGGER.md) y [benchmark.md](docs/benchmark.md).

- [x] Grafo multi-lang + anti-alucinación  
- [x] L1/L2 negocio + index incremental  
- [x] C++ / Rust / Solidity a nivel estático alto  
- [x] Testgen Python con fix loop y `skipped_env`  
- [ ] Testgen + runners nativos C++/Rust/Solidity  
- [ ] Data-flow multi-lang en subgrafo  
- [ ] Packs de framework ampliados  

---

## Creador

- GitHub: [StevAL2000](https://github.com/StevAL2000)
- LinkedIn: [Steven Alexander Rios Bonilla](https://www.linkedin.com/in/steven-alexander-rios-bonilla-919a73319)

---

## Guía de capturas (qué fotografiar)

Crea la carpeta `docs/images/` y sube PNG (o WebP) con estos nombres. Resolución recomendada: **1920×1080** o la ventana completa del navegador sin recortar barras del SO si no hace falta.

| Archivo | Qué debe verse en la foto |
|---------|---------------------------|
| `01-ui-overview.png` | Pantalla completa: sidebar Dual-RAG + chat “Sistema listo” + mapa con esfera **Listo** (como tu captura actual) |
| `02-project-indexed.png` | Tras indexar un repo real: varias esferas de archivos y enlaces; status **System Ready** |
| `03-diagnosis-evidence.png` | Una pregunta respondida: resumen estructurado, confianza, lista de relaciones (CALLS/CONTAINS/…) |
| `04-testgen-result.png` | Resultado de generar prueba (código + estado `passed` o `skipped_env` visible) |
| `05-business-layers.png` | Respuesta a “flujo de negocio” con pipeline / actores / contenedores L1–L2 |
| `06-sidebar-projects.png` | Zoom al menú izquierdo: selector de grafo, botones Usar / grafo / todo, ruta y **Cargar / indexar** |
| `07-molecular-map-evidence.png` | Mapa a pantalla casi completa con nodos iluminados tras una consulta (no solo “Listo”) |

**Tips**

- Tema oscuro (como en la UI actual) se ve mejor en GitHub.
- Oculta datos sensibles (API keys, rutas personales) o usa un repo demo.
- Si aún no tienes la foto, el README mostrará el icono roto de imagen hasta que subas el archivo — es normal.

Cuando tengas el video de YouTube, sustituye el placeholder del inicio por el enlace o el badge con miniatura.
