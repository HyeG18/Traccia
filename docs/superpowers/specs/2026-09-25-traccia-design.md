# Traccia — App de Seguimiento de Hábitos

**Fecha:** 2026-09-25
**Versión:** 0.1.0 (MVP)
**Stack:** Expo + React Native + WatermelonDB

---

## 1. Concepto y Visión

Traccia (italiano: "rastro", "seguimiento") es una app minimalista de seguimiento de hábitos con foco inicial en nutrición. Captura la esencia de "trazar tu camino" — cada entrada es un punto en tu gráfica de vida. La app transmite calma y claridad: nada de bloat, nada de métricas innecesarias. Solo vos, tus metas, y tu progreso visible.

Experiencia emocional: sentirse en control de tu día, ver tu consistencia reflejada, recibir feedback visual sin ser invasivo.

---

## 2. Design Language

### Paleta de Colores

| Rol | Color | Hex |
|-----|-------|-----|
| Primary | Verde Kirby medio | `#4CAF50` |
| Primary Dark | Verde oscuro | `#388E3C` |
| Secondary | Coral/Naranja | `#FF7043` |
| Background Dark | Gris azulado | `#1E1E2E` |
| Background Light | Crema suave | `#F5F5F0` |
| Surface Dark | Gris medio | `#2D2D3D` |
| Surface Light | Blanco suave | `#FFFFFF` |
| Text Primary Dark | Blanco | `#FFFFFF` |
| Text Primary Light | Gris oscuro | `#1A1A1A` |
| Text Secondary | Gris medio | `#757575` |
| Error | Rojo | `#EF5350` |
| Success | Verde brillante | `#66BB6A` |

### Tipografía

- **Font Family:** System default (San Francisco en iOS, Roboto en Android)
- **Heading 1:** 28px, Bold
- **Heading 2:** 22px, SemiBold
- **Body:** 16px, Regular
- **Caption:** 12px, Regular
- **Label:** 14px, Medium

### Espaciado

Base unit: 4px. Sistema de spacing: 4, 8, 12, 16, 24, 32, 48px.

### Motion

- Transiciones suaves: 200-300ms ease-out
- Micro-interacciones en botones: scale 0.97 en press
- Progress bars: animación de llenado 400ms ease-in-out
- Calendario: fade in de indicadores 150ms

---

## 3. Layout y Estructura

### Navegación

Bottom tab navigation con 4 tabs:

1. **Hoy** — pantalla principal del día actual
2. **Calendario** — vista mensual con indicadores
3. **Notas** — lista de notas/pensamientos
4. **Ajustes** — configuración de metas y preferencias

### Pantallas

#### 3.1 Hoy (Home)

- Header: fecha actual, racha actual ("🔥 X días")
- Card de progreso del día: barra circular o linear con totales vs. meta
  - Calorías (grande, central)
  - Proteína, Carbs, Grasas (tres mini-bars o números)
- Lista de comidas del día: cada entrada muestra nombre, hora, calorías
- FAB (Floating Action Button) para agregar comida
- Quick action para nota rápida (icono en header)

#### 3.2 Agregar Comida (Modal)

- Campo: nombre de comida
- Campos numéricos: calorías, proteína (gr), carbohidratos (gr), grasas (gr)
- Botón: guardar
- Botón: cancelar

#### 3.3 Calendario

- Vista mensual (calendario grid)
- Días con meta cumplida: indicador verde (dot o color de fondo)
- Días sin registrar o incompletos: sin indicador
- Tap en día: muestra resumen de ese día (sin opción de editar — solo lectura)
- Racha visible como texto cerca del header ("Racha actual: X días")

#### 3.4 Notas

- Lista cronológica (más reciente primero)
- Cada nota: texto preview + timestamp
- FAB para nueva nota
- Al tocar nota: expande a vista completa con opción de eliminar

#### 3.5 Agregar/Editar Nota (Modal)

- Campo de texto multilínea
- Timestamp automático (created_at)
- Botón guardar/cancelar

#### 3.6 Ajustes (Settings)

- Sección: Metas Diarias
  - Calorías (input numérico)
  - Proteína (gr)
  - Carbohidratos (gr)
  - Grasas (gr)
- Sección: Preferencias
  - Tema: Claro / Oscuro / Sistema
- Sección: Acerca de
  - Versión de app
  - Nombre: Traccia

---

## 4. Features

### 4.1 Seguimiento Nutricional

- Agregar comida con macros
- Editar comida existente (tap en item de la lista)
- Eliminar comida (swipe o long-press con confirmación)
- Ver totales del día actual actualizados en tiempo real

### 4.2 Metas Configurables

- Valores default: 1600kcal, 100gr proteína, 200gr carbs, 53gr grasas
- Guardadas en AsyncStorage (persistentes)
- Validación: valores numéricos positivos

### 4.3 Calendario con Indicadores

- Cada día del mes muestra si la meta fue cumplida
- Criterio de "día balanceado": todas las metas cumplidas (cada macro dentro de ±10% del objetivo)
- Navegación entre meses (swipe o flechas)
- Indicador visual: círculo verde bajo el número del día

### 4.4 Racha

- Se cuenta desde el primer día con registro
- Un día sin registrar rompe la racha
- Se muestra en pantalla Hoy y en Calendario
- Racha actual: número grande con emoji 🔥

### 4.5 Notas

- Crear nota rápida con timestamp
- Ver notas anteriores
- Eliminar nota con confirmación
- Sin límite de caracteres visible

### 4.6 Tema Claro/Oscuro

- Toggle en Ajustes
- También puede seguir "Sistema"
- Cambio inmediato, sin reload

### 4.7 Offline-First

- Todos los datos en WatermelonDB local
- Sin necesidad de conexión a internet
- Sin backend en MVP

---

## 5. Data Model (WatermelonDB)

### Table: meals

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | string | UUID, primary key |
| name | string | Nombre de la comida |
| calories | number | Kilocalorías |
| protein | number | Gramos de proteína |
| carbs | number | Gramos de carbohidratos |
| fat | number | Gramos de grasa |
| date | number | Timestamp del día (start of day) |
| created_at | number | Timestamp de creación |
| updated_at | number | Timestamp de actualización |

### Table: notes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | string | UUID, primary key |
| content | string | Texto de la nota |
| created_at | number | Timestamp de creación |

### Table: settings

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | string | "user_settings" (single row) |
| calorie_goal | number | Meta de kcal |
| protein_goal | number | Meta de proteína |
| carbs_goal | number | Meta de carbs |
| fat_goal | number | Meta de grasa |
| theme | string | "light" / "dark" / "system" |

---

## 6. Arquitectura

### Stack Tecnológico

- **Framework:** Expo SDK 52+
- **UI:** React Native (vanilla) + StyleSheet
- **Navegación:** Expo Router
- **Base de datos:** WatermelonDB
- **Almacenamiento de settings:** AsyncStorage
- **State management:** React hooks (useState, useEffect, useContext)

### Estructura de Carpetas

```
Traccia/
├── app/                    # Expo Router (rutas)
│   ├── _layout.tsx         # Root layout con providers
│   ├── index.tsx           # Hoy (Home)
│   ├── calendar.tsx       # Calendario
│   ├── notes.tsx           # Lista de notas
│   ├── settings.tsx        # Ajustes
│   └── +modal/             # Modales
│       ├── add-meal.tsx
│       └── add-note.tsx
├── components/             # Componentes reutilizables
│   ├── MealItem.tsx
│   ├── NoteItem.tsx
│   ├── ProgressCard.tsx
│   ├── DayIndicator.tsx
│   └── FAB.tsx
├── database/               # WatermelonDB
│   ├── index.ts           # DB setup
│   ├── schema.ts          # Definición de tablas
│   └── models/            # Modelos
│       ├── Meal.ts
│       ├── Note.ts
│       └── Setting.ts
├── hooks/                 # Custom hooks
│   ├── useMeals.ts
│   ├── useNotes.ts
│   ├── useSettings.ts
│   └── useTheme.ts
├── theme/                 # Temas y colores
│   └── index.ts
├── utils/                 # Utilidades
│   ├── date.ts
│   └── calculations.ts
├── docs/                  # Documentación
│   └── specs/
│       └── 2026-09-25-traccia-design.md
└── assets/                # Logo, iconos
```

### Flujo de Datos

1. UI llama hook (`useMeals`, `useNotes`, etc.)
2. Hook usa WatermelonDB observers para datos reactivos
3. Cambios en DB propagan automáticamente a UI
4. Settings en AsyncStorage (síncrono para inicialización rápida)

---

## 7. MVP Scope (v0.1.0)

### Incluido

- Agregar/editar/eliminar comida del día
- Ver progreso del día (totales vs. meta)
- Metas configurables en Ajustes
- Calendario con indicadores de días cumplidos
- Racha visible
- Agregar/ver/eliminar notas
- Tema claro/oscuro
- Persistencia local con WatermelonDB

### Excluido (futuras versiones)

- Biblioteca de alimentos (guardar comidas recurrentes)
- Sincronización cloud (Supabase)
- Notificaciones/reminders
- Gráficos de tendencia (líneas temporales)
- Módulo de ejercicio
- Módulo de lectura
- Exportación de datos
- Auth/usuarios

---

## 8. Criterios de Éxito (v0.1.0)

- App arranca sin errores en Expo Go (Android)
- Puedo agregar una comida y aparece en la lista del día
- Los totales se actualizan al instante tras agregar comida
- Al configurar metas, el progreso las refleja
- Puedo cambiar entre tema claro y oscuro
- Los datos persisten tras cerrar y reabrir la app
- Calendario muestra indicador en días con registro
- Racha se calcula y muestra correctamente
- Puedo agregar y eliminar notas

---

## 9. Logo

**Descripción para diseño externo:**

Tres a cinco círculos pequeños conectados por una línea que asciende de izquierda a derecha. El efecto final es una línea graph/tendencial que sube. Cada punto representa un día con hábito cumplido. La línea que los conecta tiene un grosor moderado. Estilo: minimalista, línea limpia, sin rellenos complejos. Los puntos son del color primary (#4CAF50). La línea del mismo tono o más oscura. Sin texto. Fondo transparente.

---

*Documento creado como parte del proceso de diseño de Traccia. Sujeto a cambios tras feedback.*
