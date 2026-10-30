<div align="center">

# 🦫 Castor en Magallanes

**Detección de impacto de *Castor canadensis* por teledetección en Tierra del Fuego**

Un detector satelital calibrado contra verdad de terreno, un visor web para explorarlo,
y un cuaderno de laboratorio que registra tanto lo que funcionó como lo que no.

<br>

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![Earth Engine](https://img.shields.io/badge/Google_Earth_Engine-API-4285F4?logo=googleearth&logoColor=white)
![MapLibre](https://img.shields.io/badge/MapLibre_GL-visor-395B8A?logo=maplibre&logoColor=white)
![Cloudflare Pages](https://img.shields.io/badge/Cloudflare_Pages-despliegue-F38020?logo=cloudflare&logoColor=white)
![Estado](https://img.shields.io/badge/estado-validaci%C3%B3n_cerrada-orange)

<br>

<img src="docs/imagenes/visor-hexagonos.jpg" alt="Visor web mostrando los diques agregados en hexágonos sobre la cobertura de MapBiomas" width="100%">

</div>

---

## Qué es esto

En 1946 se soltaron **20 castores norteamericanos (10 parejas)** en Tierra del Fuego para fomentar
la industria peletera. No tienen depredadores en el hemisferio sur. Hoy se estiman entre
**70.000 y 110.000 individuos** y unas **23.000 ha** de bosque nativo invadido.
<sub>(Universidad de Chile / Proyecto GEF Castor, Ministerio del Medio Ambiente)</sub>

Este repositorio contiene el intento de responder dos preguntas con imágenes satelitales:

| | Pregunta | Producto |
|:--|:--|:--|
| **Q1** | ¿Hay castores aquí? | Capa de candidatos con score de confianza |
| **Q2** | ¿Cuándo llegaron? | Ráster de primer año de detección |

Todo está calibrado contra **806 diques y 241 polígonos de impacto digitalizados en terreno**
sobre 130,67 km² de la comuna de Timaukel.

---

## El resultado, sin adornos

> **El detector funciona dentro de una escena. No funciona entre escenas.**

<table>
<tr><td width="50%" valign="top">

### ✅ Lo que funciona

- **F1 = 0,603** en un hold-out espacial ciego
  (Sector B, nunca usado para ajustar nada)
- **15–20×** de separación contra un control
  negativo a 1.500 km del frente de invasión
- Transfiere a otro paisaje (Isla Navarino)
- La firma espectral es real y medible

</td><td width="50%" valign="top">

### ❌ Lo que no funciona

- El nivel absoluto **no es comparable entre años**:
  varía hasta **4,5×** donde el impacto real no cambió
- Tres formulaciones del índice, tres rechazos:
  **0,603 → 0,502 → 0,473**
- El archivo Landsat tiene un hueco de **10 años**
  (1987–1996) que impide datar la llegada

</td></tr>
</table>

**El diagnóstico.** Los tres intentos revelan una tensión estructural, no un problema de
parámetros: un índice estable entre años debe promediar observaciones, y promediar borra el
detalle espacial que la detección necesita. El impacto mediano son **1,97 ha** —unos 200
píxeles de Sentinel-2— y no sobrevive al suavizado.

**El hallazgo de fondo.** El modelo conceptual del que partió el proyecto —*el castor crea agua
nueva persistente, detectable por índices de agua*— **no describe lo observado**. Tres
mediciones independientes lo contradicen. La firma real es de **biomasa seca y abierta**: SWIR
alto, NBR bajo, y una trayectoria de cobertura **Bosque → Matorral → Estepa**.

📄 El informe completo está en **[`docs/INFORME.pdf`](docs/INFORME.pdf)** (15 páginas).
El cuaderno de laboratorio, con los 22 hallazgos y los errores propios incluidos, en
**[`docs/hallazgos_fase1.md`](docs/hallazgos_fase1.md)**.

---

## El visor

Un sitio estático, sin servidor de tiles y sin llamadas a Earth Engine en tiempo de petición:
todo se precalcula en el batch y se publica como GeoJSON y PNG versionados.

<div align="center">
<img src="docs/imagenes/visor-diques.jpg" alt="Visor al nivel de dique individual" width="85%">
</div>

**Agregación progresiva con H3.** A 806 puntos juntos no se lee nada, así que los diques se
agregan en hexágonos que se relevan por zoom:

| Zoom | Capa | Celdas |
|:--|:--|:--|
| ≤ 11,6 | Hexágonos H3 res 7 (~5,2 km²) | 37 |
| 11,6 – 13,2 | Hexágonos H3 res 8 (~0,74 km²) | 144 |
| ≥ 13,2 | Diques individuales | 806 |

**Lo demás que incluye:**

- Fondo de **cobertura MapBiomas Chile Col. 2** (1999–2024) recortado a la comuna
- Visor de **4 paneles NDVI sincronizados** (2017, 2018, 2022, 2025)
- Trayectoria de cobertura dentro del impacto: el bosque cae de **59 % a 9 %** entre 1999 y 2018
- Separabilidad espectral medida (AUC) de nueve índices
- Láminas cartográficas de **CIREN · Ministerio de Agricultura** descargables en PDF

---

## Estructura

```
castor/
├── batch/                     # Pipeline. Nada de esto corre en producción.
│   ├── castor/
│   │   ├── eeinit.py          # Auth GEE + helpers de colección
│   │   ├── truth.py           # Ingesta y QA de la verdad de terreno
│   │   ├── indices.py         # NDVI, NBR, MNDWI, NDWI
│   │   ├── habitat.py         # Máscara de hábitat (MERIT Hydro + DEM)
│   │   ├── firma.py           # Separabilidad AUC impacto vs fondo
│   │   ├── score.py           # Índice compuesto y curva PR
│   │   ├── holdout.py         # Validación espacial ciega
│   │   ├── negativo.py        # Control negativo (Chiloé)
│   │   ├── serie.py           # Serie temporal Landsat 1984–2024
│   │   └── …                  # 20 módulos en total
│   ├── exportar_web.py        # GeoJSON + métricas para el visor
│   ├── exportar_cobertura.py  # Fondo MapBiomas a dos escalas
│   ├── exportar_hexagonos.py  # Agregación H3 de los diques
│   ├── exportar_ndvi.py       # Paneles NDVI multitemporales
│   └── informe_pdf.py         # Markdown → PDF con portada
│
├── config/sites.yaml          # Bloque de parámetros ÚNICO y compartido
│
├── web/                       # Lo que se sube a Cloudflare Pages
│   ├── index.html
│   ├── app.js                 # MapLibre, capas, gráficos SVG
│   ├── style.css
│   ├── _headers               # Caché: HTML revalida, assets inmutables
│   ├── datos/                 # GeoJSON, PNG y metricas.json
│   └── documentos/            # Láminas CIREN en PDF
│
└── docs/
    ├── INFORME.pdf            # Informe técnico, 15 páginas
    ├── hallazgos_fase1.md     # Cuaderno de laboratorio, 22 secciones
    └── SUBIR_A_CLOUDFLARE.md  # Pasos de despliegue
```

---

## Puesta en marcha

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Las credenciales **nunca** viven en el repositorio ni llegan a Cloudflare. `.env` solo guarda
rutas, y el `.gitignore` lo bloquea como segunda barrera:

```bash
cp .env.example .env     # apunta a C:\Users\<usuario>\.castor\
```

Ejecutar cualquier etapa del batch:

```bash
.venv\Scripts\python.exe -m batch.exportar_hexagonos
```

El visor se sirve en local sin construir nada:

```bash
python -m http.server 8799 --directory web
```

---

## Fuentes de datos

| Fuente | Uso |
|:--|:--|
| **Sentinel-2 MSI L2A** · Copernicus | Detección, índices espectrales, NDVI |
| **Landsat 5/7/8/9 Collection 2** · USGS | Serie temporal 1984–2024, datación |
| **MapBiomas Chile, Colección 2** | Cobertura y uso de suelo 1999–2024 |
| **MERIT Hydro** + **Copernicus DEM GLO-30** | Máscara de hábitat (acumulación, HAND, pendiente) |
| **JRC Global Surface Water** | Descartada: solo 0,8–1,2 % de cobertura útil aquí |
| **OpenStreetMap** (ODbL) | Límite comunal de Timaukel |
| **CIREN · Ministerio de Agricultura** | Láminas cartográficas 1:20.000 |

---

## Algunos hallazgos que vale la pena leer

Varios son resultados negativos, y están documentados con el mismo detalle que los positivos.

- **§3 — El impacto de castor no se ve como agua.** MNDWI, el índice de agua, resultó ser el
  *peor* discriminador de los nueve evaluados.
- **§4 — MapBiomas confirma la trayectoria.** Bosque → Matorral → Estepa. Y «humedal» queda
  **depletado** (0,20×), no enriquecido: en este paisaje el fondo ya es 40 % humedal natural,
  así que clasificar algo como humedal lo hace *menos* probable de ser castor.
- **§10 — Hueco de diez años en Landsat** (1987–1996): escenas que nunca se adquirieron por
  falta de estación receptora en el extremo austral.
- **§12 — JRC Global Surface Water es inutilizable aquí**, y con eso se retira una conclusión
  que estuvo a punto de publicarse.
- **§16 — Navarino no era el problema de nubosidad; Chiloé sí.**
- **§22 — El patrón de los tres intentos es el verdadero hallazgo:** la detección de objetos
  se degrada de forma monótona con el promediado, mientras la discriminación entre sitios
  mejora. Los criterios pre-registrados pedían a un solo índice dos trabajos incompatibles.

---

## Estado

El proyecto está **detenido tras la fase de validación**. El detector no superó los criterios
de aceptación fijados *antes* de correr las pruebas, y esos criterios no se movieron después
para hacerlo pasar. La propuesta de arquitectura en dos etapas que sale del diagnóstico queda
documentada en el informe, sin implementar.

El visor web sí está terminado y operativo.

---

<div align="center">
<sub>

Datos de terreno digitalizados en Tierra del Fuego · Comuna de Timaukel, Región de Magallanes
y de la Antártica Chilena, Chile

</sub>
</div>
