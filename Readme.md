# Caso 1 — Estimación de costos por materias primas (MVP)

> **Resumen ejecutivo.** Construimos un **índice de costo por equipo** a partir de X, Y, Z; generamos **P50 a 36 meses** y **bandas P10–P50–P90** por **Monte Carlo**. Entregamos un presupuesto defendible con **contingencia** explícita.

---

## 📌 Índice

- [Entregables](#entregables)
- [1) Explicación del caso](#1-explicación-del-caso)
- [2) Supuestos](#2-supuestos)
- [3) Formas de resolver y opción tomada](#3-formas-de-resolver-y-opción-tomada)
- [4) Resultados del análisis de los datos y los modelos](#4-resultados-del-análisis-de-los-datos-y-los-modelos)
  - [4.1 EDA (datos de entrada)](#41-eda-datos-de-entrada)
  - [4.2 Pronóstico y riesgo (36 meses)](#42-pronóstico-y-riesgo-36-meses)
  - [4.3 Recomendación de presupuesto (MVP)](#43-recomendación-de-presupuesto-mvp)
- [5) Futuros ajustes o mejoras](#5-futuros-ajustes-o-mejoras)
- [🔁 Reproducibilidad](#-reproducibilidad)
- [📂 Estructura sugerida del repo](#-estructura-sugerida-del-repo)
- [📋 Notas para la evaluación](#-notas-para-la-evaluación)

---

## Entregables

**Código funcional**
- `EDA_presentation.ipynb` → inventario de datos, alineación temporal, outliers, correlaciones, sensibilidad de mezcla.  
- `Code_presentation.ipynb` → costo histórico por equipo, **P50** a 36 meses, **Monte Carlo** (P10–P50–P90), exportes CSV.

**Informe** (este documento) con los títulos solicitados.

**Archivos de salida (CSV):**  
`equipo1_mc_p10_p50_p90_36m.csv`, `equipo2_mc_p10_p50_p90_36m.csv`, `series_pronosticadas_XYZ_36m.csv`, `costos_p50_36m.csv`.

---

## 1) Explicación del caso

Se requiere estimar el costo de **dos equipos** a **36 meses**. Cada costo depende de las materias primas **X, Y, Z** con **mezclas fijas** por equipo.  
La propuesta construye un **índice de costo por equipo**, obtiene un **pronóstico base (P50)** y cuantifica la **incertidumbre** mediante **simulación Monte Carlo** (bandas **P10–P50–P90**) para entregar un **presupuesto defendible** y una **contingencia** explícita.

---

## 2) Supuestos

- **Frecuencia:** mensual. **Moneda:** precio técnico (sin IVA ni aranceles).  
- **Mezclas por equipo:**

| Equipo   | Mezcla |
|---------|--------|
| Equipo 1 | 20% X + 80% Y |
| Equipo 2 | ⅓ X + ⅓ Y + ⅓ Z |

- **Componente no-material (α):** 0 en este MVP (puede habilitarse para mano de obra, logística, etc.).  
- **Alineación temporal:** índice maestro mensual + **forward-fill** (el último precio se mantiene hasta nueva observación).  
- **Modelado:** P50 **univariado** por materia prima (ARIMA/naive-drift) y **Monte Carlo** con **covarianza histórica de retornos** (P10–P50–P90).

---

## 3) Formas de resolver y opción tomada

**De lo simple a lo avanzado**

1. Precio estático del mes actual.  
2. **Indexación compuesta** por materias primas.  
3. **Pronóstico univariado (P50)** de X, Y, Z + combinación por equipo.  
4. **Monte Carlo** con correlación X–Y–Z (P10–P50–P90).  
5. *(Futuro)* FX, fletes y **markup** por proveedor.  
6. *(Futuro)* **Optimización** de abastecimiento (timing, lotes, cobertura).

> **Opción adoptada (MVP):** **2 + 3 + 4** por equilibrio entre **trazabilidad**, **rapidez** e **incertidumbre cuantificada**.

---

## 4) Resultados del análisis de los datos y los modelos

### 4.1 EDA (datos de entrada)

**Cobertura (inicio → fin; n):**

| Serie | Inicio      | Fin         | n   |
|------|-------------|-------------|-----|
| X    | 1988-06-01  | 2024-04-01  | 431 |
| Y    | 2006-01-01  | 2024-04-01  | 220 |
| Z    | 2010-01-01  | 2024-04-01  | 172 |

**Volatilidad de retornos (σ mensual):** X ≈ 0.098 · **Y ≈ 0.111** · Z ≈ 0.057  
**Correlaciones de retornos:** X–Z ≈ **0.37** (moderada), X–Y ≈ 0.05 (baja), Y–Z ≈ −0.07 (ligera opuesta).  
**Outliers (regla IQR):** X=4, **Y=67**, Z=5 (se documentan; sin recorte en el MVP).

> **Decisión EDA:** usar **alineación + forward-fill** para maximizar cobertura y **Monte Carlo** con covarianza histórica (dada la dependencia X–Z).

---

### 4.2 Pronóstico y riesgo (36 meses)

**Equipo 1 (20% X + 80% Y)**

| Métrica | Valor |
|---|---|
| P50 inicial → final | 453.72 → 442.24 (**−2.53%** en 36m) |
| Anchura relativa media | (P90−P10)/P50 ≈ **1.229** (*abanico muy ancho*) |
| Contingencia media sugerida | **(P90−P50) ≈ 356.37** (≈ **80%** del P50) |
| Mes de mayor riesgo relativo | **2026-08** |

**Equipo 2 (⅓ X + ⅓ Y + ⅓ Z)**

| Métrica | Valor |
|---|---|
| P50 inicial → final | 934.96 → 982.59 (**+5.09%** en 36m) |
| Anchura relativa media | (P90−P10)/P50 ≈ **0.540** (*riesgo moderado*) |
| Contingencia media sugerida | **(P90−P50) ≈ 299.11** (≈ **31%** del P50) |
| Mes de mayor riesgo relativo | **2026-08** |

> **Nota técnica (P50 determinista):** `costos_p50_36m.csv` puede contener **NaN** por arranque/cola desfasados en `X_fc`, `Y_fc`, `Z_fc`. **Corrección:** reindexar + *ffill* los pronósticos antes de combinarlos. Para el **MVP**, usamos el **P50 de Monte Carlo** (mediana de trayectorias), sin huecos.

---

### 4.3 Recomendación de presupuesto (MVP)

- **Base mensual:** **P50**.  
- **Colchón / contingencia:** **(P90 − P50)** promedio mensual por equipo.

| Equipo   | Contingencia sugerida |
|---------|------------------------|
| Equipo 1 | **+356** |
| Equipo 2 | **+299** |

- **Riesgo calendario:** vigilar **2026-H2** (mayores anchos relativos).

---

## 5) Futuros ajustes o mejoras

- Corregir **NaN** en P50: reindexar/*ffill* en `series_pronosticadas_XYZ_36m.csv` antes de combinar.  
- Incluir **FX** y **fletes** (si aplica); estimar **markup** por proveedor con histórico de cotizaciones.  
- **Backtest** (walk-forward) para MAPE/WAPE del P50.  
- **Winsorizar** retornos de Y (pctl 1–99) si se busca acotar bandas sin perder señal.  
- **Stress tests** (shocks) y **sensibilidad de pesos** (±10%).  
- Job **mensual** reproducible (entorno, versiones, CI/CD) y tablero/BI.

---

## 🔁 Reproducibilidad

```bash
# 1) Activar entorno (ejemplo con conda)
conda activate caso_py312

# 2) Ejecutar EDA (genera EDA_resumen.json)
# Abrir y correr: EDA_presentation.ipynb

# 3) Ejecutar pipeline de modelado
# Abrir y correr: Code_presentation.ipynb
# Exporta: series_pronosticadas_XYZ_36m.csv, costos_p50_36m.csv,
#          equipo1_mc_p10_p50_p90_36m.csv, equipo2_mc_p10_p50_p90_36m.csv
