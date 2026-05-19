# Arquitectura y Requerimientos — App Android para Validar Rentabilidad de Viajes

## Objetivo del Proyecto

Desarrollar una aplicación Android que funcione como asistente para conductores de plataformas como Uber y DiDi.

La aplicación debe:

1. Detectar automáticamente la información del viaje mostrada en pantalla.
2. Extraer datos relevantes del viaje.
3. Calcular si el viaje es rentable según reglas configuradas por el conductor.
4. Mostrar una alerta visual indicando si el viaje es rentable o no.

---

# Problema que Resuelve

Los conductores reciben solicitudes de viaje muy rápidamente y deben decidir en pocos segundos si aceptarlas.

Actualmente esta decisión se toma manualmente observando:

- Distancia de recogida
- Distancia total
- Tiempo estimado
- Pago ofrecido
- Zona del viaje
- Tráfico
- Consumo de combustible

La aplicación automatiza este análisis en tiempo real.

---

# Restricciones Importantes

Android NO permite leer directamente datos internos de Uber o DiDi mediante APIs privadas.

La app NO debe:

- Hackear aplicaciones
- Interceptar tráfico cifrado
- Modificar aplicaciones
- Automatizar aceptación de viajes

La solución debe basarse en herramientas permitidas por Android.

---

# Enfoque Correcto

## Accessibility Service

Android permite crear servicios que:

- Observan texto visible en pantalla
- Detectan cambios de interfaz
- Analizan contenido mostrado al usuario

---

# Arquitectura General

```text
┌──────────────────────────┐
│ Uber / DiDi Driver App   │
└─────────────┬────────────┘
              │
              ▼
┌──────────────────────────┐
│ Accessibility Service    │
│ Captura texto visible    │
└─────────────┬────────────┘
              │
              ▼
┌──────────────────────────┐
│ OCR / UI Parser          │
│ Extrae datos del viaje   │
└─────────────┬────────────┘
              │
              ▼
┌──────────────────────────┐
│ Motor de Reglas          │
│ Calcula rentabilidad     │
└─────────────┬────────────┘
              │
              ▼
┌──────────────────────────┐
│ Overlay flotante         │
│ Muestra resultado        │
└──────────────────────────┘
```

---

# Tecnologías Recomendadas

| Componente | Tecnología |
|---|---|
| Lenguaje | Kotlin |
| UI | Jetpack Compose |
| OCR | Google ML Kit |
| Persistencia | Room |
| Arquitectura | Clean Architecture + MVVM |
| Inyección de Dependencias | Hilt |
| Async | Coroutines |

---

# Flujo de Funcionamiento

## 1. El conductor recibe una solicitud

Uber o DiDi muestran:

- Pago
- Distancia
- Tiempo
- Destino

## 2. Accessibility Service detecta cambios

La app escucha cambios de interfaz:

```kotlin
TYPE_WINDOW_CONTENT_CHANGED
```

## 3. Extracción de datos

Ejemplo:

```text
Pago: $18.500
Recogida: 1.2 km
Destino: 8.4 km
Tiempo: 24 min
```

## 4. Normalización

```json
{
  "fare": 18500,
  "pickup_km": 1.2,
  "trip_km": 8.4,
  "minutes": 24
}
```

---

# Motor de Rentabilidad

## Parámetros configurables

### Rentabilidad mínima por km

```text
Mínimo: $1.500 COP/km
```

### Máxima distancia de recogida

```text
Máximo: 2 km
```

### Ganancia mínima total

```text
Mínimo: $10.000 COP
```

### Tiempo máximo permitido

```text
Máximo: 30 min
```

---

# Fórmulas Principales

```text
Rentabilidad = Pago Total / Distancia Total
```

```text
Distancia Total = Distancia de Recogida + Distancia del Viaje
```

---

# Overlay Flotante

La app mostrará una pequeña ventana encima de Uber/Didi.

Ejemplo:

```text
🟢 RENTABLE

$2.300/km
Recogida: 1.1 km
Tiempo: 18 min
```

---

# OCR vs Lectura Directa

## Método 1 — Accessibility Nodes

Ventajas:

- Más rápido
- Consume menos batería
- Más preciso

Problema:

- Uber/Didi pueden cambiar interfaces

## Método 2 — OCR

Ventajas:

- Más flexible

Problemas:

- Más batería
- Más lento
- Más errores

---

# Estrategia Recomendada

## Arquitectura híbrida

1. Intentar lectura por Accessibility Nodes
2. Si falla:
   - OCR automático

---

# Estructura del Proyecto

```text
app/
 ├── accessibility/
 ├── ocr/
 ├── rules/
 ├── overlay/
 ├── settings/
 ├── domain/
 ├── data/
 ├── ui/
 └── utils/
```

---

# Base de Datos

## Tabla: user_rules

```sql
id
min_profit_per_km
max_pickup_distance
min_total_fare
max_trip_time
```

---

# Permisos Android

```xml
SYSTEM_ALERT_WINDOW
BIND_ACCESSIBILITY_SERVICE
FOREGROUND_SERVICE
```

---

# Consideraciones de Rendimiento

La app debe:

- Consumir poca batería
- Evitar OCR constante
- Procesar solo cuando haya cambios
- No analizar la pantalla continuamente

---

# Roadmap MVP

## Fase 1

- Accessibility Service
- Lectura de texto
- Extracción de pago y distancia
- Overlay rentable/no rentable

## Fase 2

- OCR híbrido
- Configuración avanzada
- Historial

## Fase 3

- Machine Learning
- Estadísticas
- Predicción de rentabilidad

---

# Consideraciones Legales

La app NO debe:

- Aceptar viajes automáticamente
- Simular toques
- Interactuar automáticamente con Uber o DiDi

Debe funcionar únicamente como:

```text
Asistente visual de análisis
```

---

# Pipeline Recomendado

```text
Accessibility
    ↓
UI Parsing
    ↓
OCR fallback
    ↓
Normalization
    ↓
Rules Engine
    ↓
Overlay UI
```

---

# Posibles Nombres de Repositorio

- fareguard-android
- rentable-driver-assistant
- ride-profit-analyzer
- trip-profit-overlay
- mobility-profit-engine
