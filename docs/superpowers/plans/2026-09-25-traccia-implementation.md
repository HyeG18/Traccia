# Traccia - Implementation Plan

**Goal:** App MVP de seguimiento de hábitos nutricionales con calendario, racha y notas.

**Architecture:** Expo + React Native con Expo Router para navegación, WatermelonDB para persistencia local reactiva, AsyncStorage para settings de usuario. UI con StyleSheet nativo, tema claro/oscuro con Context.

**Tech Stack:** Expo SDK 52+, React Native, Expo Router, WatermelonDB, AsyncStorage, React Context

**Spec:** `docs/superpowers/specs/2026-09-25-traccia-design.md`

## Global Constraints

- WatermelonDB v0.27+ (compatibilidad con Expo)
- Expo Router v4+ (file-based routing)
- Node 18+ (para WatermelonDB)
- Target: Android (Expo Go inicialmente)
- No backend/cloud en MVP

---

## Task 1: Project Setup

**Files:**
- Create: `package.json`, `app.json`, `tsconfig.json`, `babel.config.js`
- Create: `theme/index.ts`
- Create: `database/schema.ts`, `database/index.ts`, `database/models/Meal.ts`, `database/models/Note.ts`

**Interfaces:**
- Produces: exports de `database`, `theme`

- [ ] **Step 1: Crear package.json**

```json
{
  "name": "traccia",
  "version": "0.1.0",
  "main": "expo-router/entry",
  "scripts": {
    "start": "expo start",
    "android": "expo start --android"
  },
  "dependencies": {
    "expo": "~52.0.0",
    "expo-router": "~4.0.0",
    "@nozbe/watermelondb": "^0.27.0",
    "@react-native-async-storage/async-storage": "1.23.1",
    "react": "18.3.1",
    "react-native": "0.76.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "@babel/core": "^7.24.0"
  }
}
```

- [ ] **Step 2: Crear app.json**

```json
{
  "expo": {
    "name": "Traccia",
    "slug": "traccia",
    "scheme": "traccia",
    "version": "0.1.0",
    "orientation": "portrait",
    "icon": "./assets/Logo.jfif",
    "userInterfaceStyle": "automatic",
    "newArchEnabled": true
  }
}
```

- [ ] **Step 3: Crear tsconfig.json**

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx"]
}
```

- [ ] **Step 4: Crear babel.config.js**

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [['@babel/plugin-proposal-decorators', { legacy: true }]],
  };
};
```

- [ ] **Step 5: Crear theme/index.ts**

```typescript
export const colors = {
  primary: '#4CAF50',
  primaryDark: '#388E3C',
  secondary: '#FF7043',
  backgroundDark: '#1E1E2E',
  backgroundLight: '#F5F5F0',
  surfaceDark: '#2D2D3D',
  surfaceLight: '#FFFFFF',
  textPrimaryDark: '#FFFFFF',
  textPrimaryLight: '#1A1A1A',
  textSecondary: '#757575',
  error: '#EF5350',
  success: '#66BB6A',
};

export const spacing = {
  xs: 4,
  sm: 8,
  md: 12,
  lg: 16,
  xl: 24,
  xxl: 32,
};

export const typography = {
  h1: { fontSize: 28, fontWeight: 'bold' as const },
  h2: { fontSize: 22, fontWeight: '600' as const },
  body: { fontSize: 16, fontWeight: 'normal' as const },
  caption: { fontSize: 12, fontWeight: 'normal' as const },
  label: { fontSize: 14, fontWeight: '500' as const },
};
```

- [ ] **Step 6: Crear database/schema.ts**

```typescript
import { appSchema, tableSchema } from '@nozbe/watermelondb';

export const schema = appSchema({
  version: 1,
  tables: [
    tableSchema({
      name: 'meals',
      columns: [
        { name: 'name', type: 'string' },
        { name: 'calories', type: 'number' },
        { name: 'protein', type: 'number' },
        { name: 'carbs', type: 'number' },
        { name: 'fat', type: 'number' },
        { name: 'date', type: 'number', isIndexed: true },
        { name: 'created_at', type: 'number' },
        { name: 'updated_at', type: 'number' },
      ],
    }),
    tableSchema({
      name: 'notes',
      columns: [
        { name: 'content', type: 'string' },
        { name: 'created_at', type: 'number' },
      ],
    }),
  ],
});
```

- [ ] **Step 7: Crear database/models/Meal.ts**

```typescript
import { Model } from '@nozbe/watermelondb';
import { field, date, readonly } from '@nozbe/watermelondb/decorators';

export default class Meal extends Model {
  static table = 'meals';

  @field('name') name!: string;
  @field('calories') calories!: number;
  @field('protein') protein!: number;
  @field('carbs') carbs!: number;
  @field('fat') fat!: number;
  @field('date') date!: number;
  @readonly @date('created_at') createdAt!: Date;
  @readonly @date('updated_at') updatedAt!: Date;
}
```

- [ ] **Step 8: Crear database/models/Note.ts**

```typescript
import { Model } from '@nozbe/watermelondb';
import { field, date, readonly } from '@nozbe/watermelondb/decorators';

export default class Note extends Model {
  static table = 'notes';

  @field('content') content!: string;
  @readonly @date('created_at') createdAt!: Date;
}
```

- [ ] **Step 9: Crear database/index.ts**

```typescript
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { schema } from './schema';
import Meal from './models/Meal';
import Note from './models/Note';

const adapter = new SQLiteAdapter({
  schema,
  jsi: true,
  onSetUpError: error => {
    console.error('Database setup error:', error);
  },
});

export const database = new Database({
  adapter,
  modelClasses: [Meal, Note],
});

export { Meal, Note };
```

- [ ] **Step 10: Commit**

```bash
cd Traccia && git init && git add -A && git commit -m "feat: project setup with Expo, WatermelonDB, theme"
```

---

## Task 2: Navigation y Layout Root

**Files:**
- Create: `app/_layout.tsx`
- Create: `hooks/useTheme.ts`, `hooks/useSettings.ts`
- Create: `components/Header.tsx`, `components/FAB.tsx`

**Interfaces:**
- Consumes: `database`, `theme`
- Produces: `ThemeProvider`, `RootLayout` con bottom tabs

- [ ] **Step 1: Crear hooks/useTheme.ts**

```typescript
import React, { createContext, useContext, useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useColorScheme } from 'react-native';
import { colors as themeColors } from '../theme';

type ThemeMode = 'light' | 'dark' | 'system';

interface ThemeContextType {
  isDark: boolean;
  mode: ThemeMode;
  setMode: (mode: ThemeMode) => void;
  colors: {
    bg: string;
    surface: string;
    text: string;
    primary: string;
    secondary: string;
    textSecondary: string;
  };
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const systemColorScheme = useColorScheme();
  const [mode, setModeState] = useState<ThemeMode>('system');

  useEffect(() => {
    AsyncStorage.getItem('theme').then(saved => {
      if (saved) setModeState(saved as ThemeMode);
    });
  }, []);

  const setMode = async (newMode: ThemeMode) => {
    setModeState(newMode);
    await AsyncStorage.setItem('theme', newMode);
  };

  const isDark = mode === 'system'
    ? systemColorScheme === 'dark'
    : mode === 'dark';

  const colors = isDark
    ? {
        bg: themeColors.backgroundDark,
        surface: themeColors.surfaceDark,
        text: themeColors.textPrimaryDark,
        primary: themeColors.primary,
        secondary: themeColors.secondary,
        textSecondary: themeColors.textSecondary,
      }
    : {
        bg: themeColors.backgroundLight,
        surface: themeColors.surfaceLight,
        text: themeColors.textPrimaryLight,
        primary: themeColors.primary,
        secondary: themeColors.secondary,
        textSecondary: themeColors.textSecondary,
      };

  return (
    <ThemeContext.Provider value={{ isDark, mode, setMode, colors }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

- [ ] **Step 2: Crear hooks/useSettings.ts**

```typescript
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export interface Settings {
  calorieGoal: number;
  proteinGoal: number;
  carbsGoal: number;
  fatGoal: number;
}

const DEFAULT_SETTINGS: Settings = {
  calorieGoal: 1600,
  proteinGoal: 100,
  carbsGoal: 200,
  fatGoal: 53,
};

export function useSettings() {
  const [settings, setSettings] = useState<Settings>(DEFAULT_SETTINGS);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    AsyncStorage.getItem('settings').then(saved => {
      if (saved) {
        setSettings({ ...DEFAULT_SETTINGS, ...JSON.parse(saved) });
      }
      setLoading(false);
    });
  }, []);

  const updateSettings = async (newSettings: Partial<Settings>) => {
    const updated = { ...settings, ...newSettings };
    setSettings(updated);
    await AsyncStorage.setItem('settings', JSON.stringify(updated));
  };

  return { settings, loading, updateSettings };
}
```

- [ ] **Step 3: Crear components/Header.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useRouter } from 'expo-router';
import { useTheme } from '../hooks/useTheme';

interface HeaderProps {
  title: string;
  streak?: number;
  showBack?: boolean;
  rightAction?: { label: string; onPress: () => void };
}

export default function Header({ title, streak, showBack, rightAction }: HeaderProps) {
  const { colors } = useTheme();
  const router = useRouter();

  return (
    <View style={[styles.container, { backgroundColor: colors.surface }]}>
      <View style={styles.left}>
        {showBack && (
          <TouchableOpacity onPress={() => router.back()}>
            <Text style={{ color: colors.text }}>←</Text>
          </TouchableOpacity>
        )}
      </View>
      <Text style={[styles.title, { color: colors.text }]}>{title}</Text>
      <View style={styles.right}>
        {streak !== undefined && (
          <Text style={styles.streak}>🔥 {streak}</Text>
        )}
        {rightAction && (
          <TouchableOpacity onPress={rightAction.onPress}>
            <Text style={{ color: colors.primary }}>{rightAction.label}</Text>
          </TouchableOpacity>
        )}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: 16,
    paddingVertical: 12,
  },
  left: { width: 60 },
  title: { fontSize: 18, fontWeight: '600' },
  right: { width: 60, flexDirection: 'row', alignItems: 'center', justifyContent: 'flex-end' },
  streak: { fontSize: 14 },
});
```

- [ ] **Step 4: Crear components/FAB.tsx**

```typescript
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';
import { useTheme } from '../hooks/useTheme';

interface FABProps {
  onPress: () => void;
}

export default function FAB({ onPress }: FABProps) {
  const { colors } = useTheme();

  return (
    <TouchableOpacity
      style={[styles.fab, { backgroundColor: colors.primary }]}
      onPress={onPress}
      activeOpacity={0.8}
    >
      <Text style={styles.icon}>+</Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  fab: {
    position: 'absolute',
    bottom: 24,
    right: 24,
    width: 56,
    height: 56,
    borderRadius: 28,
    alignItems: 'center',
    justifyContent: 'center',
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
  },
  icon: { fontSize: 28, color: 'white', fontWeight: '300' },
});
```

- [ ] **Step 5: Crear app/_layout.tsx**

```typescript
import React from 'react';
import { Tabs } from 'expo-router';
import { ThemeProvider, useTheme } from '../hooks/useTheme';

function TabLayout() {
  const { colors } = useTheme();

  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarStyle: { backgroundColor: colors.surface, borderTopColor: colors.surface },
        tabBarActiveTintColor: colors.primary,
        tabBarInactiveTintColor: colors.textSecondary,
      }}
    >
      <Tabs.Screen name="index" options={{ title: 'Hoy', tabBarIcon: () => '📅' }} />
      <Tabs.Screen name="calendar" options={{ title: 'Calendario', tabBarIcon: () => '📆' }} />
      <Tabs.Screen name="notes" options={{ title: 'Notas', tabBarIcon: () => '📝' }} />
      <Tabs.Screen name="settings" options={{ title: 'Ajustes', tabBarIcon: () => '⚙️' }} />
    </Tabs>
  );
}

export default function RootLayout() {
  return (
    <ThemeProvider>
      <TabLayout />
    </ThemeProvider>
  );
}
```

- [ ] **Step 6: Commit**

```bash
git add -A && git commit -m "feat: add navigation layout with theme provider"
```

---

## Task 3: Pantalla Hoy

**Files:**
- Create: `app/index.tsx`
- Create: `components/ProgressCard.tsx`, `components/MealItem.tsx`, `components/MacroBar.tsx`
- Create: `hooks/useMeals.ts`
- Create: `utils/date.ts`

**Interfaces:**
- Consumes: `database`, `useSettings`
- Produces: Lista de meals del día, totales, progreso

- [ ] **Step 1: Crear utils/date.ts**

```typescript
export function getStartOfDay(date: Date = new Date()): number {
  const d = new Date(date);
  d.setHours(0, 0, 0, 0);
  return d.getTime();
}

export function formatDate(date: Date): string {
  return date.toLocaleDateString('es-ES', {
    weekday: 'long',
    day: 'numeric',
    month: 'long',
  });
}

export function isSameDay(ts1: number, ts2: number): boolean {
  const d1 = new Date(ts1);
  const d2 = new Date(ts2);
  return (
    d1.getFullYear() === d2.getFullYear() &&
    d1.getMonth() === d2.getMonth() &&
    d1.getDate() === d2.getDate()
  );
}
```

- [ ] **Step 2: Crear hooks/useMeals.ts**

```typescript
import { useState, useEffect } from 'react';
import { Q } from '@nozbe/watermelondb';
import { database, Meal } from '../database';
import { getStartOfDay } from '../utils/date';

export function useMeals(date: Date = new Date()) {
  const [meals, setMeals] = useState<Meal[]>([]);
  const [loading, setLoading] = useState(true);

  const dayStart = getStartOfDay(date);

  useEffect(() => {
    const mealsCollection = database.get<Meal>('meals');
    const subscription = mealsCollection
      .query(Q.where('date', dayStart))
      .observe()
      .subscribe(result => {
        setMeals(result);
        setLoading(false);
      });

    return () => subscription.unsubscribe();
  }, [dayStart]);

  return { meals, loading };
}

export function useMealTotals(date: Date = new Date()) {
  const { meals } = useMeals(date);

  return meals.reduce(
    (acc, meal) => ({
      calories: acc.calories + meal.calories,
      protein: acc.protein + meal.protein,
      carbs: acc.carbs + meal.carbs,
      fat: acc.fat + meal.fat,
    }),
    { calories: 0, protein: 0, carbs: 0, fat: 0 }
  );
}
```

- [ ] **Step 3: Crear components/MacroBar.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

interface MacroBarProps {
  label: string;
  current: number;
  goal: number;
  unit: string;
  color: string;
}

export default function MacroBar({ label, current, goal, unit, color }: MacroBarProps) {
  const percentage = Math.min((current / goal) * 100, 100);

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.label}>{label}</Text>
        <Text style={styles.values}>
          {current}{unit} / {goal}{unit}
        </Text>
      </View>
      <View style={styles.barBackground}>
        <View style={[styles.barFill, { width: `${percentage}%`, backgroundColor: color }]} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { marginVertical: 8 },
  header: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 4 },
  label: { fontSize: 12, fontWeight: '500' },
  values: { fontSize: 12, color: '#757575' },
  barBackground: { height: 8, backgroundColor: '#E0E0E0', borderRadius: 4, overflow: 'hidden' },
  barFill: { height: '100%', borderRadius: 4 },
});
```

- [ ] **Step 4: Crear components/ProgressCard.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import { useSettings } from '../hooks/useSettings';
import { useMealTotals } from '../hooks/useMeals';
import MacroBar from './MacroBar';

export default function ProgressCard() {
  const { colors } = useTheme();
  const { settings } = useSettings();
  const totals = useMealTotals();

  const caloriePercentage = Math.min((totals.calories / settings.calorieGoal) * 100, 100);

  return (
    <View style={[styles.container, { backgroundColor: colors.surface }]}>
      <View style={styles.calorieSection}>
        <Text style={[styles.calorieValue, { color: colors.primary }]}>
          {totals.calories}
        </Text>
        <Text style={styles.calorieLabel}>kcal</Text>
        <View style={styles.calorieBar}>
          <View
            style={[
              styles.calorieBarFill,
              { width: `${caloriePercentage}%`, backgroundColor: colors.primary },
            ]}
          />
        </View>
        <Text style={styles.calorieGoal}>de {settings.calorieGoal} kcal</Text>
      </View>

      <View style={styles.macros}>
        <MacroBar label="Proteína" current={totals.protein} goal={settings.proteinGoal} unit="g" color="#4CAF50" />
        <MacroBar label="Carbos" current={totals.carbs} goal={settings.carbsGoal} unit="g" color="#FF7043" />
        <MacroBar label="Grasas" current={totals.fat} goal={settings.fatGoal} unit="g" color="#757575" />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { borderRadius: 16, padding: 16, margin: 16 },
  calorieSection: { alignItems: 'center', marginBottom: 16 },
  calorieValue: { fontSize: 48, fontWeight: 'bold' },
  calorieLabel: { fontSize: 16, color: '#757575' },
  calorieBar: { width: '100%', height: 8, backgroundColor: '#E0E0E0', borderRadius: 4, marginVertical: 8 },
  calorieBarFill: { height: '100%', borderRadius: 4 },
  calorieGoal: { fontSize: 12, color: '#757575' },
  macros: { gap: 4 },
});
```

- [ ] **Step 5: Crear components/MealItem.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import Meal from '../database/models/Meal';

interface MealItemProps {
  meal: Meal;
  onPress: () => void;
}

export default function MealItem({ meal, onPress }: MealItemProps) {
  const { colors } = useTheme();

  return (
    <TouchableOpacity
      style={[styles.container, { backgroundColor: colors.surface }]}
      onPress={onPress}
    >
      <View style={styles.left}>
        <Text style={[styles.name, { color: colors.text }]}>{meal.name}</Text>
        <Text style={styles.time}>
          {new Date(meal.createdAt).toLocaleTimeString('es-ES', { hour: '2-digit', minute: '2-digit' })}
        </Text>
      </View>
      <Text style={[styles.calories, { color: colors.primary }]}>
        {meal.calories} kcal
      </Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderRadius: 12,
    marginHorizontal: 16,
    marginVertical: 4,
  },
  left: { flex: 1 },
  name: { fontSize: 16, fontWeight: '500' },
  time: { fontSize: 12, color: '#757575', marginTop: 2 },
  calories: { fontSize: 16, fontWeight: '600' },
});
```

- [ ] **Step 6: Crear app/index.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet, FlatList } from 'react-native';
import { useRouter } from 'expo-router';
import { useTheme } from '../hooks/useTheme';
import { useMeals } from '../hooks/useMeals';
import { formatDate } from '../utils/date';
import Header from '../components/Header';
import ProgressCard from '../components/ProgressCard';
import MealItem from '../components/MealItem';
import FAB from '../components/FAB';

export default function TodayScreen() {
  const { colors } = useTheme();
  const router = useRouter();
  const { meals } = useMeals();

  return (
    <View style={[styles.container, { backgroundColor: colors.bg }]}>
      <Header title="Hoy" />
      <Text style={[styles.date, { color: colors.text }]}>{formatDate(new Date())}</Text>

      <ProgressCard />

      <Text style={[styles.sectionTitle, { color: colors.text }]}>Comidas</Text>
      <FlatList
        data={meals}
        keyExtractor={item => item.id}
        renderItem={({ item }) => (
          <MealItem meal={item} onPress={() => {}} />
        )}
        contentContainerStyle={styles.list}
        ListEmptyComponent={
          <Text style={styles.empty}>No has registrado ninguna comida hoy</Text>
        }
      />

      <FAB onPress={() => router.push('/modal/add-meal')} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  date: { fontSize: 14, marginHorizontal: 16, marginBottom: 8 },
  sectionTitle: { fontSize: 18, fontWeight: '600', marginHorizontal: 16, marginTop: 16 },
  list: { paddingBottom: 100 },
  empty: { textAlign: 'center', color: '#757575', marginTop: 32 },
});
```

- [ ] **Step 7: Commit**

```bash
git add -A && git commit -m "feat: add Today screen with progress card and meal list"
```

---

## Task 4: Modal Agregar Comida

**Files:**
- Create: `app/modal/add-meal.tsx`
- Create: `app/+modal.tsx` (modal container)

**Interfaces:**
- Consumes: `database`
- Produces: Nueva entrada en tabla `meals`

- [ ] **Step 1: Crear app/+modal.tsx**

```typescript
import { Stack } from 'expo-router';

export default function ModalLayout() {
  return <Stack screenOptions={{ presentation: 'modal' }} />;
}
```

- [ ] **Step 2: Crear app/modal/add-meal.tsx**

```typescript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  StyleSheet,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
} from 'react-native';
import { useRouter } from 'expo-router';
import { useTheme } from '../../hooks/useTheme';
import { database } from '../../database';
import { getStartOfDay } from '../../utils/date';

export default function AddMealModal() {
  const router = useRouter();
  const { colors } = useTheme();

  const [name, setName] = useState('');
  const [calories, setCalories] = useState('');
  const [protein, setProtein] = useState('');
  const [carbs, setCarbs] = useState('');
  const [fat, setFat] = useState('');

  const handleSave = async () => {
    if (!name.trim()) return;

    await database.write(async () => {
      await database.get('meals').create(meal => {
        meal.name = name.trim();
        meal.calories = parseFloat(calories) || 0;
        meal.protein = parseFloat(protein) || 0;
        meal.carbs = parseFloat(carbs) || 0;
        meal.fat = parseFloat(fat) || 0;
        meal.date = getStartOfDay();
      });
    });

    router.back();
  };

  return (
    <KeyboardAvoidingView
      style={[styles.container, { backgroundColor: colors.bg }]}
      behavior={Platform.OS === 'ios' ? 'padding' : undefined}
    >
      <View style={[styles.header, { backgroundColor: colors.surface }]}>
        <TouchableOpacity onPress={() => router.back()}>
          <Text style={{ color: colors.text }}>Cancelar</Text>
        </TouchableOpacity>
        <Text style={[styles.title, { color: colors.text }]}>Agregar Comida</Text>
        <TouchableOpacity onPress={handleSave}>
          <Text style={{ color: colors.primary, fontWeight: '600' }}>Guardar</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.form}>
        <View style={styles.field}>
          <Text style={[styles.label, { color: colors.text }]}>Nombre</Text>
          <TextInput
            style={[styles.input, { backgroundColor: colors.surface, color: colors.text }]}
            value={name}
            onChangeText={setName}
            placeholder="ej. Almuerzo"
            placeholderTextColor="#757575"
          />
        </View>

        <View style={styles.field}>
          <Text style={[styles.label, { color: colors.text }]}>Calorías (kcal)</Text>
          <TextInput
            style={[styles.input, { backgroundColor: colors.surface, color: colors.text }]}
            value={calories}
            onChangeText={setCalories}
            keyboardType="numeric"
            placeholder="0"
            placeholderTextColor="#757575"
          />
        </View>

        <View style={styles.row}>
          <View style={[styles.field, { flex: 1, marginRight: 8 }]}>
            <Text style={[styles.label, { color: colors.text }]}>Proteína (g)</Text>
            <TextInput
              style={[styles.input, { backgroundColor: colors.surface, color: colors.text }]}
              value={protein}
              onChangeText={setProtein}
              keyboardType="numeric"
              placeholder="0"
              placeholderTextColor="#757575"
            />
          </View>
          <View style={[styles.field, { flex: 1, marginLeft: 8 }]}>
            <Text style={[styles.label, { color: colors.text }]}>Carbos (g)</Text>
            <TextInput
              style={[styles.input, { backgroundColor: colors.surface, color: colors.text }]}
              value={carbs}
              onChangeText={setCarbs}
              keyboardType="numeric"
              placeholder="0"
              placeholderTextColor="#757575"
            />
          </View>
        </View>

        <View style={styles.field}>
          <Text style={[styles.label, { color: colors.text }]}>Grasas (g)</Text>
          <TextInput
            style={[styles.input, { backgroundColor: colors.surface, color: colors.text }]}
            value={fat}
            onChangeText={setFat}
            keyboardType="numeric"
            placeholder="0"
            placeholderTextColor="#757575"
          />
        </View>
      </View>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#E0E0E0',
  },
  title: { fontSize: 18, fontWeight: '600' },
  form: { padding: 16 },
  field: { marginBottom: 16 },
  label: { fontSize: 14, fontWeight: '500', marginBottom: 8 },
  input: {
    padding: 12,
    borderRadius: 8,
    fontSize: 16,
  },
  row: { flexDirection: 'row' },
});
```

- [ ] **Step 3: Commit**

```bash
git add -A && git commit -m "feat: add meal modal with form"
```

---

## Task 5: Pantalla Calendario

**Files:**
- Create: `app/calendar.tsx`
- Create: `components/DayIndicator.tsx`
- Create: `utils/calculations.ts`

**Interfaces:**
- Consumes: `useMeals`, `useSettings`
- Produces: Vista mensual con indicadores

- [ ] **Step 1: Crear utils/calculations.ts**

```typescript
import { getStartOfDay } from './date';

interface DayTotals {
  calories: number;
  protein: number;
  carbs: number;
  fat: number;
}

interface Settings {
  calorieGoal: number;
  proteinGoal: number;
  carbsGoal: number;
  fatGoal: number;
}

export function isDayBalanced(totals: DayTotals, settings: Settings): boolean {
  const tolerance = 0.1;
  const check = (current: number, goal: number) =>
    current >= goal * (1 - tolerance) && current <= goal * (1 + tolerance);

  return (
    check(totals.calories, settings.calorieGoal) &&
    check(totals.protein, settings.proteinGoal) &&
    check(totals.carbs, settings.carbsGoal) &&
    check(totals.fat, settings.fatGoal)
  );
}
```

- [ ] **Step 2: Crear components/DayIndicator.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useTheme } from '../hooks/useTheme';

interface DayIndicatorProps {
  day: number;
  isBalanced: boolean;
  isToday: boolean;
  onPress: () => void;
}

export default function DayIndicator({ day, isBalanced, isToday, onPress }: DayIndicatorProps) {
  const { colors } = useTheme();

  return (
    <TouchableOpacity style={styles.container} onPress={onPress}>
      <View
        style={[
          styles.dayCircle,
          isToday && { borderWidth: 2, borderColor: colors.primary },
          isBalanced && { backgroundColor: colors.primary },
        ]}
      >
        <Text
          style={[
            styles.dayText,
            { color: isBalanced ? 'white' : colors.text },
          ]}
        >
          {day}
        </Text>
      </View>
      {isBalanced && <View style={[styles.dot, { backgroundColor: colors.primary }]} />}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  container: { alignItems: 'center', padding: 4 },
  dayCircle: {
    width: 36,
    height: 36,
    borderRadius: 18,
    alignItems: 'center',
    justifyContent: 'center',
  },
  dayText: { fontSize: 14 },
  dot: { width: 4, height: 4, borderRadius: 2, marginTop: 2 },
});
```

- [ ] **Step 3: Crear app/calendar.tsx**

```typescript
import React, { useState, useMemo } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import Header from '../components/Header';
import DayIndicator from '../components/DayIndicator';

const DAYS = ['Dom', 'Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb'];
const MONTHS = ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre'];

export default function CalendarScreen() {
  const { colors } = useTheme();
  const [currentDate, setCurrentDate] = useState(new Date());

  const year = currentDate.getFullYear();
  const month = currentDate.getMonth();

  const daysInMonth = useMemo(() => {
    const firstDay = new Date(year, month, 1).getDay();
    const numDays = new Date(year, month + 1, 0).getDate();
    const days: (number | null)[] = [];

    for (let i = 0; i < firstDay; i++) days.push(null);
    for (let i = 1; i <= numDays; i++) days.push(i);

    return days;
  }, [year, month]);

  const today = new Date();
  const isToday = (day: number) =>
    day === today.getDate() && month === today.getMonth() && year === today.getFullYear();

  const goToPreviousMonth = () => {
    setCurrentDate(new Date(year, month - 1, 1));
  };

  const goToNextMonth = () => {
    setCurrentDate(new Date(year, month + 1, 1));
  };

  return (
    <View style={[styles.container, { backgroundColor: colors.bg }]}>
      <Header title="Calendario" />

      <View style={styles.monthNav}>
        <TouchableOpacity onPress={goToPreviousMonth}>
          <Text style={{ color: colors.text }}>←</Text>
        </TouchableOpacity>
        <Text style={[styles.monthTitle, { color: colors.text }]}>
          {MONTHS[month]} {year}
        </Text>
        <TouchableOpacity onPress={goToNextMonth}>
          <Text style={{ color: colors.text }}>→</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.weekDays}>
        {DAYS.map(day => (
          <Text key={day} style={[styles.weekDay, { color: colors.textSecondary }]}>
            {day}
          </Text>
        ))}
      </View>

      <View style={styles.calendar}>
        {daysInMonth.map((day, index) => (
          <View key={index} style={styles.dayCell}>
            {day !== null && (
              <DayIndicator
                day={day}
                isBalanced={false}
                isToday={isToday(day)}
                onPress={() => {}}
              />
            )}
          </View>
        ))}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  monthNav: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 16,
    marginBottom: 16,
  },
  monthTitle: { fontSize: 18, fontWeight: '600' },
  weekDays: {
    flexDirection: 'row',
    paddingHorizontal: 8,
    marginBottom: 8,
  },
  weekDay: { flex: 1, textAlign: 'center', fontSize: 12, fontWeight: '500' },
  calendar: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    paddingHorizontal: 8,
  },
  dayCell: { width: '14.28%', aspectRatio: 1, padding: 4 },
});
```

- [ ] **Step 4: Commit**

```bash
git add -A && git commit -m "feat: add calendar screen with day indicators"
```

---

## Task 6: Pantalla Notas

**Files:**
- Create: `app/notes.tsx`
- Create: `components/NoteItem.tsx`
- Create: `hooks/useNotes.ts`
- Create: `app/modal/add-note.tsx`

**Interfaces:**
- Consumes: `database`
- Produces: CRUD de notas

- [ ] **Step 1: Crear hooks/useNotes.ts**

```typescript
import { useState, useEffect } from 'react';
import { Q } from '@nozbe/watermelondb';
import { database, Note } from '../database';

export function useNotes() {
  const [notes, setNotes] = useState<Note[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const notesCollection = database.get<Note>('notes');
    const subscription = notesCollection
      .query(Q.sortBy('created_at', Q.desc))
      .observe()
      .subscribe(result => {
        setNotes(result);
        setLoading(false);
      });

    return () => subscription.unsubscribe();
  }, []);

  return { notes, loading };
}
```

- [ ] **Step 2: Crear components/NoteItem.tsx**

```typescript
import React, { useState } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import Note from '../database/models/Note';

interface NoteItemProps {
  note: Note;
  onPress: () => void;
  onDelete: () => void;
}

export default function NoteItem({ note, onPress, onDelete }: NoteItemProps) {
  const { colors } = useTheme();
  const [showDelete, setShowDelete] = useState(false);

  const formatDate = (date: Date) => {
    return date.toLocaleDateString('es-ES', {
      day: 'numeric',
      month: 'short',
      hour: '2-digit',
      minute: '2-digit',
    });
  };

  return (
    <TouchableOpacity
      style={[styles.container, { backgroundColor: colors.surface }]}
      onPress={onPress}
      onLongPress={() => setShowDelete(true)}
    >
      <Text style={[styles.content, { color: colors.text }]} numberOfLines={3}>
        {note.content}
      </Text>
      <Text style={styles.date}>{formatDate(note.createdAt)}</Text>
      {showDelete && (
        <TouchableOpacity style={styles.deleteBtn} onPress={onDelete}>
          <Text style={{ color: '#EF5350' }}>Eliminar</Text>
        </TouchableOpacity>
      )}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  container: { padding: 16, borderRadius: 12, marginHorizontal: 16, marginVertical: 4 },
  content: { fontSize: 14, lineHeight: 20 },
  date: { fontSize: 12, color: '#757575', marginTop: 8 },
  deleteBtn: { position: 'absolute', top: 8, right: 8, padding: 4 },
});
```

- [ ] **Step 3: Crear app/notes.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet, FlatList, Alert } from 'react-native';
import { useRouter } from 'expo-router';
import { useTheme } from '../hooks/useTheme';
import { useNotes } from '../hooks/useNotes';
import { database } from '../database';
import Header from '../components/Header';
import NoteItem from '../components/NoteItem';
import FAB from '../components/FAB';

export default function NotesScreen() {
  const { colors } = useTheme();
  const router = useRouter();
  const { notes } = useNotes();

  const handleDelete = (note: any) => {
    Alert.alert('Eliminar nota', '¿Estás seguro?', [
      { text: 'Cancelar', style: 'cancel' },
      {
        text: 'Eliminar',
        style: 'destructive',
        onPress: async () => {
          await database.write(async () => {
            await note.destroyPermanently();
          });
        },
      },
    ]);
  };

  return (
    <View style={[styles.container, { backgroundColor: colors.bg }]}>
      <Header title="Notas" />
      <FlatList
        data={notes}
        keyExtractor={item => item.id}
        renderItem={({ item }) => (
          <NoteItem note={item} onPress={() => {}} onDelete={() => handleDelete(item)} />
        )}
        contentContainerStyle={styles.list}
        ListEmptyComponent={
          <Text style={styles.empty}>No tienes notas todavía</Text>
        }
      />
      <FAB onPress={() => router.push('/modal/add-note')} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  list: { paddingVertical: 8, paddingBottom: 100 },
  empty: { textAlign: 'center', color: '#757575', marginTop: 32 },
});
```

- [ ] **Step 4: Crear app/modal/add-note.tsx**

```typescript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  StyleSheet,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
} from 'react-native';
import { useRouter } from 'expo-router';
import { useTheme } from '../../hooks/useTheme';
import { database } from '../../database';

export default function AddNoteModal() {
  const router = useRouter();
  const { colors } = useTheme();
  const [content, setContent] = useState('');

  const handleSave = async () => {
    if (!content.trim()) return;

    await database.write(async () => {
      await database.get('notes').create(note => {
        note.content = content.trim();
      });
    });

    router.back();
  };

  return (
    <KeyboardAvoidingView
      style={[styles.container, { backgroundColor: colors.bg }]}
      behavior={Platform.OS === 'ios' ? 'padding' : undefined}
    >
      <View style={[styles.header, { backgroundColor: colors.surface }]}>
        <TouchableOpacity onPress={() => router.back()}>
          <Text style={{ color: colors.text }}>Cancelar</Text>
        </TouchableOpacity>
        <Text style={[styles.title, { color: colors.text }]}>Nueva Nota</Text>
        <TouchableOpacity onPress={handleSave}>
          <Text style={{ color: colors.primary, fontWeight: '600' }}>Guardar</Text>
        </TouchableOpacity>
      </View>

      <TextInput
        style={[styles.input, { backgroundColor: colors.surface, color: colors.text }]}
        value={content}
        onChangeText={setContent}
        placeholder="Escribe tu nota..."
        placeholderTextColor="#757575"
        multiline
        autoFocus
      />
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#E0E0E0',
  },
  title: { fontSize: 18, fontWeight: '600' },
  input: {
    flex: 1,
    padding: 16,
    fontSize: 16,
    textAlignVertical: 'top',
  },
});
```

- [ ] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: add notes screen with CRUD"
```

---

## Task 7: Pantalla Ajustes

**Files:**
- Create: `app/settings.tsx`

**Interfaces:**
- Consumes: `useSettings`, `useTheme`
- Produces: UI para editar metas y tema

- [ ] **Step 1: Crear app/settings.tsx**

```typescript
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity, ScrollView, Switch } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import { useSettings } from '../hooks/useSettings';
import Header from '../components/Header';

export default function SettingsScreen() {
  const { colors, isDark, setMode } = useTheme();
  const { settings, updateSettings } = useSettings();

  const renderGoalInput = (
    label: string,
    value: number,
    unit: string,
    key: 'calorieGoal' | 'proteinGoal' | 'carbsGoal' | 'fatGoal'
  ) => (
    <View style={styles.goalRow}>
      <Text style={[styles.goalLabel, { color: colors.text }]}>{label}</Text>
      <View style={styles.goalInputWrapper}>
        <TouchableOpacity
          style={[styles.goalBtn, { backgroundColor: colors.surface }]}
          onPress={() => updateSettings({ [key]: Math.max(0, value - 10) })}
        >
          <Text style={{ color: colors.text }}>-</Text>
        </TouchableOpacity>
        <Text style={[styles.goalValue, { color: colors.text }]}>
          {value}{unit}
        </Text>
        <TouchableOpacity
          style={[styles.goalBtn, { backgroundColor: colors.surface }]}
          onPress={() => updateSettings({ [key]: value + 10 })}
        >
          <Text style={{ color: colors.text }}>+</Text>
        </TouchableOpacity>
      </View>
    </View>
  );

  return (
    <ScrollView style={[styles.container, { backgroundColor: colors.bg }]}>
      <Header title="Ajustes" />

      <Text style={[styles.sectionTitle, { color: colors.text }]}>Metas Diarias</Text>
      <View style={[styles.section, { backgroundColor: colors.surface }]}>
        {renderGoalInput('Calorías', settings.calorieGoal, ' kcal', 'calorieGoal')}
        {renderGoalInput('Proteína', settings.proteinGoal, ' g', 'proteinGoal')}
        {renderGoalInput('Carbohidratos', settings.carbsGoal, ' g', 'carbsGoal')}
        {renderGoalInput('Grasas', settings.fatGoal, ' g', 'fatGoal')}
      </View>

      <Text style={[styles.sectionTitle, { color: colors.text }]}>Apariencia</Text>
      <View style={[styles.section, { backgroundColor: colors.surface }]}>
        <View style={styles.themeRow}>
          <Text style={{ color: colors.text }}>Modo Oscuro</Text>
          <Switch
            value={isDark}
            onValueChange={value => setMode(value ? 'dark' : 'light')}
            trackColor={{ false: '#E0E0E0', true: colors.primary }}
          />
        </View>
      </View>

      <Text style={[styles.sectionTitle, { color: colors.text }]}>Acerca de</Text>
      <View style={[styles.section, { backgroundColor: colors.surface }]}>
        <Text style={{ color: colors.text }}>Traccia v0.1.0</Text>
        <Text style={{ color: colors.textSecondary, marginTop: 4 }}>
          Trace your journey
        </Text>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  sectionTitle: { fontSize: 14, fontWeight: '600', marginHorizontal: 16, marginTop: 24, marginBottom: 8 },
  section: { marginHorizontal: 16, borderRadius: 12, padding: 16 },
  goalRow: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', marginBottom: 16 },
  goalLabel: { fontSize: 16 },
  goalInputWrapper: { flexDirection: 'row', alignItems: 'center' },
  goalBtn: { width: 36, height: 36, borderRadius: 18, alignItems: 'center', justifyContent: 'center' },
  goalValue: { fontSize: 16, fontWeight: '600', minWidth: 80, textAlign: 'center' },
  themeRow: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' },
});
```

- [ ] **Step 2: Commit**

```bash
git add -A && git commit -m "feat: add settings screen with goals and theme toggle"
```

---

## Task 8: Integración y Verificación

**Files:**
- Verificar todos los archivos creados

- [ ] **Step 1: Verificar estructura**

```bash
find . -name "*.ts" -o -name "*.tsx" | sort
```

- [ ] **Step 2: Instalar dependencias**

```bash
npm install
```

- [ ] **Step 3: Generar proyecto Android**

```bash
npx expo prebuild --platform android
```

- [ ] **Step 4: Commit final**

```bash
git add -A && git commit -m "chore: final integration"
```

---

## Self-Review Checklist

- [ ] Spec coverage: Todos los requisitos del spec tienen tarea
- [ ] Placeholder scan: Sin "TBD", "TODO", o pasos vagos
- [ ] Type consistency: Nombres de funciones consistentes entre tasks
- [ ] Task boundaries: Cada task produce algo testeable independientemente
- [ ] WatermelonDB decorators: `@field`, `@date` correctos
- [ ] Expo Router: File-based routing seguido correctamente
- [ ] Theme: Context provider envuelve toda la app
- [ ] Offline-first: Sin dependencias de red en MVP

---

## Orden de Ejecución

1. Task 1: Project Setup
2. Task 2: Navigation y Layout
3. Task 3: Pantalla Hoy
4. Task 4: Modal Agregar Comida
5. Task 5: Calendario
6. Task 6: Notas
7. Task 7: Ajustes
8. Task 8: Integración y Verificación
