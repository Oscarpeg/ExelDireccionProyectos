# "Modelo Predictivo de Aprobación de Préstamos: Un Análisis de Clasificación Supervisada para Evaluación de Riesgo Crediticio"

---

## Introducción

La evaluación del riesgo crediticio representa uno de los pilares fundamentales en la gestión financiera de las instituciones bancarias. La capacidad de predecir si un solicitante de préstamo cumplirá con sus obligaciones de pago determina directamente la rentabilidad y estabilidad de la cartera crediticia.

Tradicionalmente, la evaluación crediticia se ha basado en las conocidas "5 C's del Crédito":
- **Character** (Carácter): Historial de cumplimiento del solicitante
- **Capacity** (Capacidad): Habilidad para pagar basada en ingresos
- **Capital**: Patrimonio y ahorros del solicitante
- **Collateral** (Colateral): Garantías ofrecidas
- **Conditions** (Condiciones): Propósito del préstamo y condiciones económicas

En un contexto donde el volumen de solicitudes crece exponencialmente y la competencia por buenos clientes se intensifica, resulta fundamental desarrollar modelos predictivos que permitan:
1. **Automatizar** la evaluación inicial de solicitudes
2. **Reducir** el riesgo de default (incumplimiento)
3. **Identificar** patrones en el comportamiento crediticio
4. **Optimizar** la toma de decisiones

Este análisis busca desarrollar un modelo de **clasificación binaria** que prediga si un préstamo será exitosamente pagado o entrará en default, utilizando técnicas de Machine Learning con Pipelines y evaluando el rendimiento mediante métricas como Precision, Recall y Accuracy.

---

## Los Datos

El dataset analizado contiene **45,000 solicitudes de préstamo** con información detallada del perfil del solicitante y características del préstamo.

### Variables del Dataset

| Variable | Tipo | Descripción |
|----------|------|-------------|
| `person_age` | Numérica | Edad del solicitante (años) |
| `person_gender` | Categórica | Género (male/female) |
| `person_education` | Categórica | Nivel educativo |
| `person_income` | Numérica | Ingresos anuales ($) |
| `person_emp_exp` | Numérica | Años de experiencia laboral |
| `person_home_ownership` | Categórica | Tipo de vivienda |
| `loan_amnt` | Numérica | Monto del préstamo solicitado ($) |
| `loan_intent` | Categórica | Propósito del préstamo |
| `loan_int_rate` | Numérica | Tasa de interés (%) |
| `loan_percent_income` | Numérica | Préstamo como % del ingreso |
| `cb_person_cred_hist_length` | Numérica | Años de historial crediticio |
| `credit_score` | Numérica | Puntaje crediticio |
| `previous_loan_defaults_on_file` | Categórica | ¿Tiene defaults previos? (Yes/No) |
| **`loan_status`** | **Objetivo** | **0 = Default, 1 = Pagado** |

El dataset permite analizar la interacción entre el perfil socioeconómico del solicitante, las características del préstamo y el resultado final del crédito.

---

## 1. ¿Cuál es la distribución de aprobaciones en el dataset?

### 1.1 Distribución de la Variable Objetivo

| Estado | Cantidad | Porcentaje |
|--------|----------|------------|
| Rechazado/Default (0) | 35,000 | 77.8% |
| Aprobado/Pagado (1) | 10,000 | 22.2% |

**HALLAZGO CRÍTICO:** El dataset presenta un **desbalance significativo** - solo el 22.2% de los préstamos son exitosos. Esto tiene implicaciones importantes:

1. Un modelo que simplemente prediga "Rechazado" para todos los casos tendría 77.8% de accuracy, pero sería inútil.
2. Las métricas como **Precision y Recall** son más informativas que Accuracy en este contexto.
3. Se requiere estratificación en la división de datos para mantener las proporciones.

### 1.2 Perfil del Solicitante Promedio

| Característica | Valor Promedio |
|----------------|----------------|
| Edad | 27.76 años |
| Ingresos anuales | $80,319 |
| Experiencia laboral | 5.41 años |
| Historial crediticio | 5.87 años |
| Credit Score | 632.61 |
| Monto solicitado | $9,583 |
| Tasa de interés | 11.01% |
| Préstamo/Ingreso | 14% |

El solicitante típico es relativamente joven (28 años), con ingresos moderados y un credit score en rango "Fair" (580-669). El préstamo promedio representa el 14% del ingreso anual.

---

## 2. ¿Qué factores influyen en la aprobación del préstamo?

### 2.1 Análisis por Género

| Género | Tasa de Aprobación |
|--------|-------------------|
| Female | 22.2% |
| Male | 22.2% |

**CONCLUSIÓN:** El género **NO** es un factor diferenciador en la aprobación de préstamos. Ambos grupos tienen exactamente la misma tasa de éxito (22.2%), lo cual es positivo desde la perspectiva de equidad crediticia.

### 2.2 Análisis por Nivel Educativo

| Nivel Educativo | Tasa de Aprobación |
|-----------------|-------------------|
| Doctorate | 22.9% |
| Bachelor | 22.5% |
| High School | 22.3% |
| Associate | 22.0% |
| Master | 21.8% |

**CONCLUSIÓN:** El nivel educativo tiene un impacto **marginal** en la aprobación. La diferencia entre el más alto (Doctorate: 22.9%) y el más bajo (Master: 21.8%) es de solo 1.1 puntos porcentuales. Sorprendentemente, los poseedores de maestría tienen la tasa más baja.

### 2.3 HALLAZGO CRÍTICO: Defaults Previos

| Defaults Previos | Total Solicitudes | Tasa de Aprobación |
|------------------|-------------------|-------------------|
| **No** | 22,142 | **45.2%** |
| **Yes** | 22,858 | **0.0%** |

**ESTE ES EL FACTOR MÁS DETERMINANTE DEL ANÁLISIS.**

- Si el solicitante **NO tiene defaults previos**: casi la mitad (45.2%) obtiene el préstamo.
- Si el solicitante **TIENE defaults previos**: la probabilidad de aprobación es **CERO**.

Esta variable por sí sola divide el dataset en dos grupos completamente distintos:
- El grupo sin defaults previos (49.2% del total) concentra el **100%** de las aprobaciones.
- El grupo con defaults previos (50.8% del total) tiene **0%** de aprobaciones.

**IMPLICACIÓN:** Un sistema de reglas simple que rechace automáticamente a solicitantes con defaults previos sería extremadamente efectivo.

### 2.4 Análisis por Propósito del Préstamo

| Propósito | Tasa de Aprobación |
|-----------|-------------------|
| DEBTCONSOLIDATION | 30.3% |
| MEDICAL | 27.8% |
| HOMEIMPROVEMENT | 26.3% |
| PERSONAL | 20.1% |
| EDUCATION | 17.0% |
| VENTURE | 14.4% |

**CONCLUSIONES:**
- **Consolidación de deuda** tiene la mayor tasa de aprobación (30.3%), posiblemente porque indica intención de ordenar finanzas.
- **Gastos médicos** también tienen alta aprobación (27.8%), considerados necesidades urgentes.
- **Venture (emprendimiento)** tiene la menor tasa (14.4%), reflejando el mayor riesgo percibido.
- **Educación** sorprendentemente baja (17.0%), posiblemente por el perfil joven de estos solicitantes.

### 2.5 Análisis por Tipo de Vivienda

| Tipo de Vivienda | % del Dataset | Tasa de Aprobación |
|------------------|---------------|-------------------|
| RENT | 52.1% | ~22% |
| MORTGAGE | 41.1% | ~22% |
| OWN | 6.6% | ~22% |
| OTHER | 0.3% | ~22% |

La mayoría de solicitantes alquilan (52%) o tienen hipoteca (41%). El tipo de vivienda no muestra diferencias significativas en la tasa de aprobación.

---

## 3. ¿Cuáles son las correlaciones más importantes?

### 3.1 Correlación con loan_status

| Variable | Correlación | Interpretación |
|----------|-------------|----------------|
| **loan_percent_income** | **+0.3849** | Mayor % del ingreso → Mayor probabilidad de default |
| **loan_int_rate** | **+0.3320** | Mayor tasa de interés → Mayor probabilidad de default |
| person_income | -0.1358 | Mayor ingreso → Menor probabilidad de default |
| loan_amnt | +0.1077 | Mayor monto → Mayor probabilidad de default |
| person_age | -0.0215 | Correlación débil |
| person_emp_exp | -0.0205 | Correlación débil |
| cb_person_cred_hist_length | -0.0149 | Correlación débil |
| credit_score | -0.0076 | Correlación muy débil |

**HALLAZGOS CLAVE:**

1. **loan_percent_income** (+0.38): El factor más predictivo. Cuando el préstamo representa un alto porcentaje del ingreso, el riesgo de default aumenta significativamente.

2. **loan_int_rate** (+0.33): Las tasas de interés altas se asocian con mayor default. Esto puede ser porque:
   - Los clientes de mayor riesgo reciben tasas más altas
   - Las tasas altas dificultan el pago

3. **credit_score** (-0.008): Sorprendentemente, el credit score tiene correlación casi nula con el resultado. Esto sugiere que otros factores (como defaults previos) tienen mayor peso predictivo.

---

## 4. ¿Qué tan bien predice el modelo?

### 4.1 Configuración del Modelo

**Pipeline de Preprocesamiento:**
- Variables numéricas: Imputación (mediana) → Escalado (StandardScaler)
- Variables categóricas: Imputación (moda) → One-Hot Encoding

**División de Datos:**
- Entrenamiento: 36,000 muestras (80%)
- Prueba: 9,000 muestras (20%)
- Estratificación para mantener proporción de clases

### 4.2 Comparación de Modelos

| Modelo | Accuracy | Precision | Recall | F1-Score | AUC-ROC |
|--------|----------|-----------|--------|----------|---------|
| **Random Forest** | **0.9274** | **0.8918** | **0.7665** | **0.8244** | **0.9741** |
| Gradient Boosting | 0.9252 | 0.8829 | 0.7650 | 0.8197 | 0.9725 |
| Logistic Regression | 0.8993 | 0.7888 | 0.7470 | 0.7673 | 0.9562 |
| KNN | 0.8956 | 0.7909 | 0.7205 | 0.7541 | 0.9246 |

**MEJOR MODELO: Random Forest**

- **Accuracy 92.74%**: De cada 100 predicciones, 93 son correctas.
- **Precision 89.18%**: De los préstamos que aprobamos, 89% efectivamente pagan.
- **Recall 76.65%**: Detectamos el 77% de los buenos clientes.
- **AUC-ROC 0.9741**: Excelente capacidad para distinguir entre buenos y malos clientes.

### 4.3 Matriz de Confusión del Mejor Modelo

|  | Predicho: Rechazado | Predicho: Aprobado |
|--|---------------------|-------------------|
| **Real: Rechazado** | 6,814 (TN) | 186 (FP) |
| **Real: Aprobado** | 467 (FN) | 1,533 (TP) |

**Interpretación detallada:**

| Métrica | Valor | Significado en contexto bancario |
|---------|-------|----------------------------------|
| **Verdaderos Negativos (TN)** | 6,814 | Préstamos correctamente rechazados - evitamos pérdidas |
| **Falsos Positivos (FP)** | 186 | Préstamos aprobados que harán default - **PÉRDIDA PARA EL BANCO** |
| **Falsos Negativos (FN)** | 467 | Buenos clientes rechazados - **OPORTUNIDAD PERDIDA** |
| **Verdaderos Positivos (TP)** | 1,533 | Préstamos correctamente aprobados - generan ganancia |

**Tasas de Error:**
- **Tasa de Falsos Positivos: 2.66%** - Solo el 2.66% de los rechazos reales fueron aprobados incorrectamente.
- **Tasa de Falsos Negativos: 23.35%** - El 23.35% de los buenos clientes fueron rechazados.

---

## 5. ¿Cómo interpretar las métricas de evaluación?

### 5.1 Definición de Métricas

| Métrica | Fórmula | Pregunta que responde |
|---------|---------|----------------------|
| **ACCURACY** | (TP + TN) / Total | ¿Qué proporción de predicciones son correctas? |
| **PRECISION** | TP / (TP + FP) | De los préstamos que aprobamos, ¿cuántos pagan? |
| **RECALL** | TP / (TP + FN) | De los buenos clientes, ¿a cuántos aprobamos? |
| **F1-SCORE** | 2×(P×R)/(P+R) | Balance entre Precision y Recall |
| **AUC-ROC** | Área bajo curva ROC | Capacidad de distinguir entre clases |

### 5.2 Trade-off Precision vs Recall en Contexto Bancario

**Escenario 1: Priorizar PRECISION (Banco conservador)**
- Objetivo: Minimizar préstamos malos aprobados
- Consecuencia: Menos defaults, pero también menos clientes
- Umbral de decisión: Alto (ej: aprobar solo si probabilidad > 70%)

**Escenario 2: Priorizar RECALL (Banco agresivo)**
- Objetivo: Capturar la mayor cantidad de buenos clientes
- Consecuencia: Más clientes, pero también más defaults
- Umbral de decisión: Bajo (ej: aprobar si probabilidad > 30%)

**Para el modelo actual:**
- Precision = 89.18%: De cada 100 préstamos aprobados, 89 son pagados (11 hacen default)
- Recall = 76.65%: De cada 100 buenos clientes, aprobamos a 77 (perdemos 23)

### 5.3 ¿Qué métrica priorizar?

En el contexto de riesgo crediticio, generalmente se prioriza **PRECISION** porque:

1. El costo de un **Falso Positivo** (aprobar un mal préstamo) es alto:
   - Pérdida del capital prestado
   - Costos de cobranza
   - Provisiones contables

2. El costo de un **Falso Negativo** (rechazar un buen cliente) es menor:
   - Solo se pierde el margen de ganancia
   - El cliente puede ir a la competencia

Sin embargo, un banco muy conservador perdería participación de mercado. El **F1-Score** ofrece un balance razonable.

---

## 6. ¿Cuáles son las variables más importantes para el modelo?

### 6.1 Ranking de Importancia (Random Forest)

| Posición | Variable | Importancia | Interpretación |
|----------|----------|-------------|----------------|
| 1 | **loan_int_rate** | 0.1504 | Tasa de interés - proxy de riesgo percibido |
| 2 | **loan_percent_income** | 0.1409 | Carga financiera relativa al ingreso |
| 3 | **previous_loan_defaults_on_file_Yes** | 0.1322 | Historial de incumplimiento |
| 4 | **previous_loan_defaults_on_file_No** | 0.1248 | Historial limpio |
| 5 | **person_income** | 0.1171 | Capacidad de pago absoluta |
| 6 | loan_amnt | 0.0584 | Monto del préstamo |
| 7 | credit_score | 0.0525 | Puntaje crediticio |
| 8 | person_home_ownership_RENT | 0.0383 | Estabilidad habitacional |
| 9 | person_age | 0.0293 | Edad del solicitante |
| 10 | person_emp_exp | 0.0268 | Estabilidad laboral |

### 6.2 Interpretación de los Factores Clave

**1. Tasa de Interés (15.04%)**
La tasa de interés es el predictor más importante. Esto refleja que:
- Los bancos asignan tasas más altas a clientes más riesgosos
- La tasa funciona como un "indicador de riesgo" ya asignado
- Tasas altas dificultan el cumplimiento de pagos

**2. Préstamo como % del Ingreso (14.09%)**
Este ratio mide la carga financiera:
- Alto porcentaje → Mayor esfuerzo para pagar → Mayor riesgo de default
- Regla general: préstamos > 30% del ingreso son riesgosos

**3. Defaults Previos (26.70% combinado)**
Las dos categorías de defaults previos (Yes/No) suman el 26.7% de importancia:
- Es el factor más determinante cuando se considera en conjunto
- Refleja el comportamiento crediticio histórico

**4. Ingresos (11.71%)**
Los ingresos absolutos importan menos que el ratio préstamo/ingreso, pero siguen siendo relevantes para evaluar la capacidad de pago.

---

## 7. ¿Qué tan robusto es el modelo?

### 7.1 Validación Cruzada (5-Fold)

| Modelo | F1-Score Promedio | Desviación Estándar |
|--------|-------------------|---------------------|
| **Random Forest** | **0.8256** | **±0.0113** |
| Gradient Boosting | 0.8186 | ±0.0123 |
| Logistic Regression | 0.7624 | ±0.0079 |
| KNN | 0.7515 | ±0.0073 |

**INTERPRETACIÓN:**
- Random Forest mantiene un F1-Score consistente de 0.82 ± 0.01 a través de diferentes particiones de datos.
- La baja desviación estándar indica que el modelo es **estable** y no depende de una partición particular de los datos.
- El rendimiento es **generalizable** a datos nuevos.

---

## 8. Conclusiones

### 8.1 FACTOR DETERMINANTE: DEFAULTS PREVIOS

El análisis revela que el historial de incumplimiento es el factor más crítico:
- **0% de aprobación** para solicitantes con defaults previos
- **45.2% de aprobación** para solicitantes sin defaults previos

**Recomendación:** Implementar un filtro automático que rechace solicitudes con defaults previos, lo cual simplificaría significativamente el proceso.

### 8.2 RENDIMIENTO DEL MODELO

El modelo Random Forest logra un rendimiento excepcional:

| Métrica | Valor | Interpretación |
|---------|-------|----------------|
| Accuracy | 92.74% | 93 de cada 100 decisiones son correctas |
| Precision | 89.18% | 89% de los préstamos aprobados son pagados |
| Recall | 76.65% | Capturamos 77% de los buenos clientes |
| AUC-ROC | 0.9741 | Excelente capacidad discriminativa |

### 8.3 VARIABLES PREDICTIVAS CLAVE

Los factores más importantes para predecir el éxito del préstamo son:
1. **Tasa de interés** - Refleja el riesgo percibido
2. **Préstamo como % del ingreso** - Mide la carga financiera
3. **Defaults previos** - Historial de comportamiento
4. **Ingresos** - Capacidad de pago

### 8.4 MÉTRICAS DE EVALUACIÓN - RESUMEN

| Contexto | Métrica Prioritaria | Razón |
|----------|---------------------|-------|
| **Banco conservador** | PRECISION | Minimizar pérdidas por default |
| **Banco agresivo** | RECALL | Capturar más clientes |
| **Balance general** | F1-SCORE | Equilibrio entre ambos objetivos |
| **Evaluación global** | AUC-ROC | Capacidad de distinguir clases |

### 8.5 USO DE PIPELINES

Los Pipelines de scikit-learn proporcionan:
- **Reproducibilidad**: Mismo preprocesamiento en entrenamiento y predicción
- **Prevención de data leakage**: El escalado se ajusta solo con datos de entrenamiento
- **Facilidad de despliegue**: Un solo objeto contiene todo el flujo de trabajo
- **Mantenibilidad**: Código más limpio y organizado

### 8.6 RECOMENDACIONES FINALES

1. **Rechazar automáticamente** solicitudes con defaults previos
2. **Evaluar cuidadosamente** préstamos que excedan el 30% del ingreso
3. **Monitorear** la tasa de interés como indicador de riesgo
4. **Ajustar el umbral de decisión** según el apetito de riesgo del banco
5. **Reentrenar periódicamente** el modelo con datos actualizados

---

*Análisis realizado sobre dataset de 45,000 solicitudes de préstamo. Metodología: Clasificación supervisada con Pipelines (Random Forest, Gradient Boosting, Logistic Regression, KNN). Métricas de evaluación: Precision, Recall, Accuracy, F1-Score, AUC-ROC.*
