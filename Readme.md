### Informe

Fecha: 2025-11-03
Código funcional: EDA_presentation.ipynb y Code_presentation.ipynb (pipeline reproducible; exporta CSV de resultados).

Entregables

Código funcional

EDA_presentation.ipynb → inventario de datos, alineación temporal, outliers, correlaciones, sensibilidad de mezcla.

Code_presentation.ipynb → costo histórico por equipo, P50 a 36 meses, Monte Carlo (P10–P50–P90), exportes CSV.

Informe (este documento) con los títulos solicitados.

Archivos de salida (CSV):
equipo1_mc_p10_p50_p90_36m.csv, equipo2_mc_p10_p50_p90_36m.csv, series_pronosticadas_XYZ_36m.csv, costos_p50_36m.csv.

1) Explicación del caso

Se requiere estimar el costo de dos equipos a 36 meses. Cada costo depende de las materias primas X, Y, Z con mezclas fijas por equipo.
La propuesta construye un índice de costo por equipo, obtiene un pronóstico base (P50) y cuantifica la incertidumbre mediante simulación Monte Carlo (bandas P10–P50–P90) para entregar un presupuesto defendible y una contingencia explícita.

2) Supuestos

Frecuencia: mensual. Moneda: precio técnico (sin IVA ni aranceles).

Mezclas por equipo:

Equipo 1: 20% X + 80% Y

Equipo 2: ⅓ X + ⅓ Y + ⅓ Z

Componente no-material (α): 0 en esta versión (se puede activar para mano de obra, logística, etc.).

Alineación temporal: índice maestro mensual + forward-fill (el último precio observado se mantiene hasta nueva actualización).

Modelado: P50 univariado por materia prima (ARIMA/naive-drift) y Monte Carlo con covarianza histórica de retornos para P10–P50–P90.

3) Formas para resolver el caso y opción tomada

De lo simple a lo avanzado:

Precio estático del mes actual.

Indexación compuesta por materias primas.

Pronóstico univariado (P50) de X, Y, Z + combinación por equipo.

Monte Carlo con correlación X–Y–Z (P10–P50–P90).

(Futuro) Modelo con FX, fletes y markup por proveedor.

(Futuro) Optimización de abastecimiento (timing, lotes, cobertura).

Opción adoptada (MVP): 2 + 3 + 4 por su equilibrio entre trazabilidad, rapidez e incertidumbre cuantificada.

4) Resultados del análisis de los datos y los modelos
4.1 EDA (datos de entrada)

Cobertura (inicio → fin; n):

X: 1988-06-01 → 2024-04-01 · n=431

Y: 2006-01-01 → 2024-04-01 · n=220

Z: 2010-01-01 → 2024-04-01 · n=172

Volatilidad de retornos (σ mensual): X ≈ 0.098 · Y ≈ 0.111 · Z ≈ 0.057

Correlaciones de retornos: X–Z ≈ 0.37 (moderada), X–Y ≈ 0.05 (baja), Y–Z ≈ −0.07 (ligera opuesta).

Outliers (regla IQR): X=4, Y=67, Z=5. Documentados; no se recortan en el MVP (winsorización opcional).

Decisión de EDA: usar alineación + forward-fill para maximizar cobertura y Monte Carlo con covarianza (dada la dependencia X–Z).

4.2 Pronóstico y riesgo (36 meses)
Equipo 1 (20% X + 80% Y)

P50 inicial → final: 453.72 → 442.24 (−2.53% en 36m).

Anchura relativa media: (P90 − P10) / P50 ≈ 1.229 (abanico muy ancho).

Contingencia media sugerida: (P90 − P50) ≈ 356.37 (≈ 80% del P50).

Mes de mayor riesgo relativo: 2026-08.

Equipo 2 (⅓ X + ⅓ Y + ⅓ Z)

P50 inicial → final: 934.96 → 982.59 (+5.09% en 36m).

Anchura relativa media: (P90 − P10) / P50 ≈ 0.540 (riesgo moderado).

Contingencia media sugerida: (P90 − P50) ≈ 299.11 (≈ 31% del P50).

Mes de mayor riesgo relativo: 2026-08.

Nota técnica (P50 determinista): costos_p50_36m.csv puede contener NaN por arranque/cola desfasados en X_fc, Y_fc, Z_fc. Corrección: reindexar + ffill los pronósticos antes de combinarlos. Para el MVP, el P50 de Monte Carlo (mediana de trayectorias) se usa como base y no presenta huecos.

4.3 Recomendación de presupuesto (MVP)

Base mensual: P50.

Colchón/contingencia: (P90 − P50) promedio mensual por equipo.

Equipo 1: +356

Equipo 2: +299

Gestión de riesgo calendario: vigilar 2026-H2 (mayores anchos relativos).

5) Futuros ajustes o mejoras

Corregir NaN en P50: reindexar/ffill en series_pronosticadas_XYZ_36m.csv antes de combinar.

Añadir FX y fletes (si aplica); estimar markup por proveedor con cotizaciones históricas.

Backtest (walk-forward) para MAPE/WAPE del P50.

Winsorizar retornos de Y (percentiles 1–99) si se busca acotar bandas sin perder señal.

Stress tests (shocks) y sensibilidad de pesos (±10%).

Job mensual reproducible (entorno, versiones, CI/CD) y tablero/BI.