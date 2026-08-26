# Dual-Rag---Debugger-
Sistema de debugging con IA local o conexión via api (con mínimo uso de tokens por consulta). El enfoque es aumentar precisión y reducir el abuso de tokens para auditar proyectos.


## Benchmark actualizado Dual-RAG Debugger  
*(excluye empaquetador / launcher · incluye C++, Rust y Solidity al nivel estático de Python)*

Escala: **A** producción útil · **B** bueno con huecos · **C** indexa, poca profundidad · **D** solo extensión · **—** no aplica.

---

### 1. Por lenguaje (indexación + grafo)

| Lenguaje | Extensiones | Extractor | Símbolos / imports / calls | Relaciones finas | Madurez |
|----------|-------------|-----------|----------------------------|------------------|---------|
| **Python** | `.py` | AST nativo + tree-sitter | clases, métodos, calls, imports | CONTAINS, CALLS, DEPENDS_ON, import→pkg | **A** |
| **C / C++** | `.c` `.cpp` `.h` `.hpp`… | **`lang_cpp`** (tree-sitter + regex rico) | namespace, class/struct/enum, métodos, includes | CONTAINS, **IMPORTS** (`#include`), **CALLS**, **INHERITS** | **A−** |
| **Rust** | `.rs` | **`lang_rust`** (tree-sitter + regex rico) | mod, use, struct/enum/**trait**, impl, fn, macro | CONTAINS, IMPORTS, CALLS, **IMPLEMENTS** | **A−** |
| **Solidity** | `.sol` | **`lang_solidity`** (tree-sitter + regex rico) | contract/interface/library, fn, modifier, event, error, pragma | CONTAINS, IMPORTS, CALLS, **INHERITS** (`is`), **EMITS** | **A−** |
| **HTML** | `.html` `.htm` | tree-sitter-html + path_refs | tags, id, class, scripts | MATCHES_SELECTOR, LINKS_TO, NESTS, assets bi-dir | **A−** |
| **CSS** | `.css` | tree-sitter-css | selectores | MATCHES_SELECTOR | **B+** |
| **JS / TS** | `.js` `.ts`… | tree-sitter + `lang_js` | functions, imports, calls | CALLS, imports | **B** |
| **Go** | `.go` | `lang_go` | funcs, imports | básicos | **B−** |
| **Shell** | `.sh` `.bash` | `lang_shell` | comandos, paths | path-ish | **B** |
| **YAML** | `.yml` `.yaml` | estructurado | keys / servicios | débil | **C+** |
| **Dockerfile** | `Dockerfile` | nombre especial | stages | básico | **C** |
| **PHP** | `.php` | tree-sitter/regex | símbolos básicos | poco framework | **C** |
| **Java / C#** | `.java` `.cs` | extract ligero | parcial | superficial | **C** |
| **SQL** | `.sql` | patrones | tablas/refs débiles | muy limitado | **C−** |
| **Otro** | — | `unsupported_refs` | detecta no soportado | no parsea | **D** (honesto) |

---

### 2. Qué cambió en C++ / Rust / Solidity

Antes: familia registrada → AST genérico o fallback pobre → pocas aristas útiles.

Ahora (módulos dedicados en `analysis/`):

| Capacidad | C++ | Rust | Solidity |
|-----------|-----|------|----------|
| Includes / use / import | `#include`, `using` | `use`, `extern crate` | `import "…"` |
| Tipos de primer nivel | class, struct, enum, union, namespace | struct, enum, **trait**, mod, type | contract, interface, library |
| Herencia / impl | **INHERITS** (`: public Base`) | **IMPLEMENTS** (`impl Trait for T`) | **INHERITS** (`is Ownable`) |
| Funciones / métodos | fn + `Class::method` | `fn`, firmas en impl | `function`, visibility, mutability |
| Extra de dominio | calls `.` / `->` | macros `!`, MODULE | **EMITS**, MODIFIER, EVENT, ERROR, PRAGMA |
| Sin tree-sitter instalado | regex rico sigue indexando | igual | igual |
| Con tree-sitter-* en venv | AST preferente | AST preferente | AST preferente |

**Pruebas locales (regex path):** Service→Base, Runnable TRAIT + IMPLEMENTS, Token is Ownable + emit Transfer.

**Routing:** `FileAnalyzer` → `CppAnalyzer` / `RustAnalyzer` / `SolidityAnalyzer` (ya no caen en el saco genérico).

---

### 3. Por objetivo de producto

| Objetivo | Estado | Nota post C++/Rust/Sol |
|----------|--------|-------------------------|
| Mapa de negocio L1/L2 | **A** | Independiente del lenguaje del repo |
| Grafo estructural L3 | **A** Python · **A−** C++/Rust/Sol · **B** web/JS · **C** resto | Subida clara de los tres |
| Deps / versiones L4 | **A−** Python · **B** Node/Go | C++/Rust/Sol: manifiesto parcial (CMake/Cargo/Foundry aún no profundo) |
| Anti-alucinación + owner | **A−** | Igual |
| Assets HTML/JS bidireccionales | **A−** | Sin cambio |
| Diagnóstico chat anclado | **A** si grafo = proyecto activo | C++/Rust/Sol ya anclan símbolos reales |
| Testgen + fix loop | **A−** Python · **B** JS/Go · **—** C++/Rust/Sol | **Siguiente hueco grande** |
| Data-flow / null | **B** Python · **—** nativos | Sin cambio |
| Index incremental | **A** | Reindex solo `.cpp`/`.rs`/`.sol` dirty |

---

### 4. Circuito “debugger” por lenguaje

| Paso | Python | **C++** | **Rust** | **Solidity** | JS/TS | Go |
|------|--------|---------|----------|--------------|-------|-----|
| Indexar + relacionar | A | **A−** | **A−** | **A−** | B | B− |
| Pregunta anclada al grafo | A | **A−** | **A−** | **A−** | B | B− |
| Flujo de negocio L1/L2 | A | A* | A* | A* | B* | B* |
| Generar test nativo | A | — | — | — | B | B |
| Ejecutar + consola real | A (pytest) | — | — | — | B (node) | B (go test) |
| Fix loop ≤2 | A− | — | — | — | B | B |

\*L1/L2 del proyecto, no del lenguaje.

---

### 5. Scores globales (actualizado)

| Dimensión | Score |
|-----------|--------|
| Debugging **Python** punta a punta | **~8.5 / 10** |
| Grafo + diagnóstico **C++ / Rust / Solidity** | **~7.5–8 / 10** (antes ~4–5) |
| Front **HTML/JS + paths** | **~8 / 10** |
| Negocio multi-carpeta (L1/L2) | **~8 / 10** |
| Testgen JS/Go | **~6.5 / 10** |
| Testgen / runtime nativo C++·Rust·Sol | **~2 / 10** (aún sin runners) |
| Resto de lenguajes | **~4–5 / 10** |
| “Mejor debugger multi-lang” | **Más cerca en estático nativo**; falta **cerrar el circuito ejecutar→arreglar** en C++/Rust/Solidity |

---


**Resumen:** en **análisis estático y anclaje de diagnóstico**, C++, Rust y Solidity ya están en la misma liga que Python (**A−**). La brecha que queda es **runtime + testgen**, no el grafo.
