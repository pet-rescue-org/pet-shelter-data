<div align="center">
  <img src="https://img.icons8.com/color/96/000000/database.png" alt="Database Icon" width="80"/>
  
  # 🗄️ PetRescue - Repositorio de Almacenamiento (Storage Bank)

  **Sistema Ligero de CMS Headless y Base de Datos Sincronizada**
  
  [![GitHub API](https://img.shields.io/badge/GitHub%20API-Storage-181717?logo=github&logoColor=white)](https://docs.github.com/en/rest)
  [![JSON Data](https://img.shields.io/badge/Data-JSON-000000?logo=json&logoColor=white)]()
  [![WebP Optimized](https://img.shields.io/badge/Images-WebP-4CAF50?logo=image&logoColor=white)]()
</div>

---

## 📌 ¿Qué es este repositorio?

Este repositorio es una pieza fundamental de la arquitectura desacoplada de **PetRescue**. No contiene código ejecutable. Funciona exclusivamente como **Base de Datos y CDN Ligero (Headless CMS)**.

La aplicación principal (Backend Vercel) se comunica con este repositorio mediante la **API REST de GitHub** para realizar operaciones CRUD (Crear, Leer, Actualizar, Borrar) sobre archivos `.json` e imágenes.

Al usar GitHub como storage:
- Mantenemos los costos de infraestructura al mínimo.
- Tenemos versionado histórico automático de cada adopción, animal o evento.
- Aprovechamos la CDN global y gratuita de GitHub (`raw.githubusercontent.com`) para servir los datos ultrarrápidos.

---

## 🌳 Árbol Simétrico de Directorios

El sistema está rigurosamente estructurado para mantener un orden perfecto entre los datos de **Mascotas** y **Anuncios**.

```text
📦 petrescue-storage
 ┣ 📂 data/                           # Entidades JSON
 ┃ ┣ 📂 announcements/                # Backup individual de cada anuncio
 ┃ ┃ ┣ 📜 announcement-12345.json
 ┃ ┃ ┗ 📜 announcement-67890.json
 ┃ ┣ 📂 pets/                         # Backup individual de cada mascota
 ┃ ┃ ┣ 📜 pet-11111.json
 ┃ ┃ ┗ 📜 pet-22222.json
 ┃ ┣ 📜 announcements.json            # 🔴 ÍNDICE MAESTRO DE EVENTOS (Array)
 ┃ ┗ 📜 pets.json                     # 🔴 ÍNDICE MAESTRO DE MASCOTAS (Array)
 ┃
 ┣ 📂 images/                         # Archivos Multimedia Optimizados (WebP)
 ┃ ┣ 📂 announcements/                # Flyers e imágenes de eventos
 ┃ ┃ ┣ 🖼️ announcement-12345.webp
 ┃ ┃ ┗ 🖼️ announcement-67890.webp
 ┃ ┗ 📂 pets/                         # Fotografías de las mascotas
 ┃   ┣ 🖼️ pet-11111.webp
 ┃   ┗ 🖼️ pet-22222.webp
 ┃
 ┗ 📜 README.md                       # Este archivo
```

---

## ♻️ Flujo del Ciclo de Vida de los Datos

El backend de **PetRescue** maneja este repositorio de forma automatizada mediante un patrón de **Persistencia Dual**:

1. **Creación/Edición (Optimizada a WebP):**
   - Cuando el Admin sube una foto, el cliente (Frontend) la convierte a formato `.webp` de alta compresión.
   - El backend sube la imagen a `images/.../{id}.webp`.
   - Se crea el archivo individual `data/.../{id}.json` (Para backup atómico).
   - Se inyecta (Upsert) el objeto en el tope (unshift) del array dentro de `data/pets.json` o `data/announcements.json`.

2. **Lectura (Lectura Rápida):**
   - La aplicación pública **solo** consume los archivos `pets.json` y `announcements.json`. Al estar en un solo archivo, evitamos múltiples peticiones a la API.

3. **Borrado Sincronizado (Limpieza Automática):**
   - Al eliminar un registro desde el panel, el backend **destruye tres elementos simultáneamente**:
     1. Elimina el archivo de imagen `.webp`.
     2. Elimina el archivo `.json` individual.
     3. Elimina el registro específico dentro del índice maestro (`pets.json` / `announcements.json`).
   - *Nota: Si la eliminación de la imagen o del JSON individual falla, un script de limpieza regular puede ser ejecutado para mantener la integridad (ej. `scripts/cleanup-announcement-images.ts`).*

---

## ⚠️ Advertencias y Recomendaciones de Edición

> [!CAUTION]
> **ESTRICTO: EDICIÓN MANUAL DESACONSEJADA**
> Este repositorio es administrado automáticamente por la API. La manipulación manual de los archivos puede romper la aplicación principal.

Si es estrictamente necesario editar manualmente, obedece estas reglas:

- **Respeta la Sintaxis JSON:** Una sola coma faltante en `pets.json` o `announcements.json` causará un Crash Total en el Frontend. Verifica la validez del JSON antes de hacer commit.
- **Mantén la Simetría:** Si borras un archivo de imagen en `images/pets/`, DEBES asegurarte de borrar su referencia en `pets.json`.
- **No renombres imágenes:** El frontend usa una regla estricta de nombres basada en el `{id}`. Si cambias el nombre de `pet-123.webp`, la app ya no la encontrará.
- **Formato WebP:** Si subes imágenes a mano, conviértelas **siempre** a formato `.webp` para mantener el ecosistema ligero y rápido.

---
<div align="center">
  <i>Generado automáticamente para la administración de datos estructurados.</i>
</div>
