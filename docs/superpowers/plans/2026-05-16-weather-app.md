# Weather App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Погодное приложение на Expo 55 с GPS, ручным поиском города, прогнозом на 3/5/7 дней и переключением тёмной/светлой темы.

**Architecture:** React Context хранит всё глобальное состояние (погода, тема, город). Три экрана соединены stack-навигацией. Два сервиса работают с Open-Meteo API — один за погодой, другой за геокодингом.

**Tech Stack:** Expo SDK 55, React Native 0.83.6, `@react-navigation/native-stack`, `expo-location`, Open-Meteo API (бесплатно, без ключа), Jest + jest-expo для тестов.

---

## Файловая структура

```
WeatherApp/
├── App.js                          ← modify: навигация + WeatherProvider
├── app.json                        ← modify: разрешения геолокации
├── package.json                    ← modify: jest конфигурация
├── constants/
│   └── themes.js                   ← create: цвета светлой/тёмной темы
├── context/
│   └── WeatherContext.js           ← create: глобальный state
├── services/
│   ├── weatherApi.js               ← create: запросы к Open-Meteo
│   ├── geocodingApi.js             ← create: поиск города по названию
│   └── __tests__/
│       ├── weatherApi.test.js      ← create: тесты сервиса погоды
│       └── geocodingApi.test.js    ← create: тесты геокодинга
├── components/
│   ├── WeatherIcon.js              ← create: эмодзи-иконка + описание
│   └── WeatherCard.js              ← create: карточка дня прогноза
└── screens/
    ├── HomeScreen.js               ← create: текущая погода
    ├── ForecastScreen.js           ← create: прогноз на несколько дней
    └── SettingsScreen.js           ← create: поиск города + тема
```

---

## Task 1: Установка зависимостей и настройка разрешений

**Files:**
- Modify: `app.json`
- Modify: `package.json`

- [ ] **Step 1: Установить библиотеки навигации**

```bash
npm install @react-navigation/native @react-navigation/native-stack
```

- [ ] **Step 2: Установить нативные зависимости навигации через Expo**

```bash
npx expo install react-native-screens react-native-safe-area-context
```

Expo сам выберет версии, совместимые с SDK 55.

- [ ] **Step 3: Установить expo-location**

```bash
npx expo install expo-location
```

- [ ] **Step 4: Установить зависимости для тестов**

```bash
npx expo install --dev jest-expo @testing-library/react-native
```

- [ ] **Step 5: Добавить jest конфигурацию в package.json**

Открой `package.json` и добавь блок `"jest"` (после `"private": true`):

```json
{
  "name": "weatherapp",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "test": "jest"
  },
  "dependencies": {
    "@react-navigation/native": "^7.0.0",
    "@react-navigation/native-stack": "^7.0.0",
    "expo": "~55.0.24",
    "expo-location": "~18.0.7",
    "expo-status-bar": "~55.0.6",
    "react": "19.2.0",
    "react-native": "0.83.6",
    "react-native-safe-area-context": "^5.0.0",
    "react-native-screens": "^4.0.0"
  },
  "devDependencies": {
    "@testing-library/react-native": "^13.0.0",
    "jest-expo": "~55.0.0"
  },
  "jest": {
    "preset": "jest-expo"
  },
  "private": true
}
```

- [ ] **Step 6: Добавить разрешения геолокации в app.json**

Открой `app.json` и добавь разрешения:

```json
{
  "expo": {
    "name": "weatherapp",
    "slug": "weatherapp",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash-icon.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "infoPlist": {
        "NSLocationWhenInUseUsageDescription": "This app uses your location to show local weather."
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": ["android.permission.ACCESS_FINE_LOCATION"]
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

- [ ] **Step 7: Установить все зависимости**

```bash
npm install
```

- [ ] **Step 8: Проверить что приложение запускается**

```bash
npx expo start
```

Ожидаемый результат: QR-код в терминале, приложение открывается с текстом "Open up App.js to start working on your app!"

- [ ] **Step 9: Закоммитить**

```bash
git add package.json app.json package-lock.json
git commit -m "feat: install navigation, location, and test dependencies"
```

---

## Task 2: Константы темы

**Files:**
- Create: `constants/themes.js`

- [ ] **Step 1: Создать папку и файл**

```bash
mkdir constants
```

- [ ] **Step 2: Написать themes.js**

Создай файл `constants/themes.js`:

```javascript
export const themes = {
  light: {
    background: '#e8f4f8',
    card: '#ffffff',
    text: '#333333',
    subtext: '#666666',
    accent: '#3498db',
    border: '#ddd',
    statusBar: 'dark',
  },
  dark: {
    background: '#1a1a2e',
    card: '#16213e',
    text: '#e0e0e0',
    subtext: '#aaaaaa',
    accent: '#4a9fd4',
    border: '#333333',
    statusBar: 'light',
  },
};
```

- [ ] **Step 3: Закоммитить**

```bash
git add constants/themes.js
git commit -m "feat: add light and dark theme constants"
```

---

## Task 3: Сервис погоды (TDD)

**Files:**
- Create: `services/weatherApi.js`
- Create: `services/__tests__/weatherApi.test.js`

- [ ] **Step 1: Создать папки**

```bash
mkdir -p services/__tests__
```

- [ ] **Step 2: Написать тесты (сначала!)**

Создай файл `services/__tests__/weatherApi.test.js`:

```javascript
import { fetchCurrentWeather, fetchForecast } from '../weatherApi';

global.fetch = jest.fn();

beforeEach(() => fetch.mockClear());

describe('fetchCurrentWeather', () => {
  it('returns current weather data on success', async () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        current: {
          temperature_2m: 20,
          apparent_temperature: 18,
          relative_humidity_2m: 65,
          windspeed_10m: 10,
          weathercode: 0,
        },
      }),
    });

    const result = await fetchCurrentWeather(52.52, 13.41);

    expect(result.temperature_2m).toBe(20);
    expect(result.weathercode).toBe(0);
  });

  it('throws when response is not ok', async () => {
    fetch.mockResolvedValueOnce({ ok: false });
    await expect(fetchCurrentWeather(52.52, 13.41)).rejects.toThrow(
      'Failed to fetch weather data'
    );
  });
});

describe('fetchForecast', () => {
  it('returns forecast data on success', async () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        daily: {
          time: ['2026-01-01', '2026-01-02'],
          temperature_2m_max: [22, 19],
          temperature_2m_min: [15, 12],
          weathercode: [1, 3],
        },
      }),
    });

    const result = await fetchForecast(52.52, 13.41, 2);

    expect(result.time).toHaveLength(2);
    expect(result.temperature_2m_max[0]).toBe(22);
  });

  it('throws when response is not ok', async () => {
    fetch.mockResolvedValueOnce({ ok: false });
    await expect(fetchForecast(52.52, 13.41, 5)).rejects.toThrow(
      'Failed to fetch forecast data'
    );
  });
});
```

- [ ] **Step 3: Запустить тесты — убедиться что они падают**

```bash
npx jest services/__tests__/weatherApi.test.js
```

Ожидаемый результат: `Cannot find module '../weatherApi'`

- [ ] **Step 4: Написать реализацию**

Создай файл `services/weatherApi.js`:

```javascript
const BASE_URL = 'https://api.open-meteo.com/v1/forecast';

export async function fetchCurrentWeather(latitude, longitude) {
  const params = new URLSearchParams({
    latitude,
    longitude,
    current: 'temperature_2m,apparent_temperature,relative_humidity_2m,windspeed_10m,weathercode',
    timezone: 'auto',
  });
  const response = await fetch(`${BASE_URL}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch weather data');
  const data = await response.json();
  return data.current;
}

export async function fetchForecast(latitude, longitude, days) {
  const params = new URLSearchParams({
    latitude,
    longitude,
    daily: 'temperature_2m_max,temperature_2m_min,weathercode',
    forecast_days: String(days),
    timezone: 'auto',
  });
  const response = await fetch(`${BASE_URL}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch forecast data');
  const data = await response.json();
  return data.daily;
}
```

- [ ] **Step 5: Запустить тесты — убедиться что они проходят**

```bash
npx jest services/__tests__/weatherApi.test.js
```

Ожидаемый результат: `4 passed`

- [ ] **Step 6: Закоммитить**

```bash
git add services/weatherApi.js services/__tests__/weatherApi.test.js
git commit -m "feat: add weather API service with tests"
```

---

## Task 4: Сервис геокодинга (TDD)

**Files:**
- Create: `services/geocodingApi.js`
- Create: `services/__tests__/geocodingApi.test.js`

- [ ] **Step 1: Написать тесты**

Создай файл `services/__tests__/geocodingApi.test.js`:

```javascript
import { searchCity } from '../geocodingApi';

global.fetch = jest.fn();

beforeEach(() => fetch.mockClear());

describe('searchCity', () => {
  it('returns city name and coordinates on success', async () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        results: [{ name: 'London', latitude: 51.5074, longitude: -0.1278 }],
      }),
    });

    const result = await searchCity('London');

    expect(result.name).toBe('London');
    expect(result.latitude).toBe(51.5074);
    expect(result.longitude).toBe(-0.1278);
  });

  it('throws when city is not found', async () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({ results: [] }),
    });

    await expect(searchCity('xyzxyzxyz')).rejects.toThrow(
      'City "xyzxyzxyz" not found'
    );
  });

  it('throws when response is not ok', async () => {
    fetch.mockResolvedValueOnce({ ok: false });
    await expect(searchCity('London')).rejects.toThrow('Failed to search for city');
  });
});
```

- [ ] **Step 2: Запустить тесты — убедиться что они падают**

```bash
npx jest services/__tests__/geocodingApi.test.js
```

Ожидаемый результат: `Cannot find module '../geocodingApi'`

- [ ] **Step 3: Написать реализацию**

Создай файл `services/geocodingApi.js`:

```javascript
const GEOCODING_URL = 'https://geocoding-api.open-meteo.com/v1/search';

export async function searchCity(name) {
  const params = new URLSearchParams({ name, count: '1', language: 'en' });
  const response = await fetch(`${GEOCODING_URL}?${params}`);
  if (!response.ok) throw new Error('Failed to search for city');
  const data = await response.json();
  if (!data.results || data.results.length === 0) {
    throw new Error(`City "${name}" not found`);
  }
  const { name: cityName, latitude, longitude } = data.results[0];
  return { name: cityName, latitude, longitude };
}
```

- [ ] **Step 4: Запустить тесты — убедиться что они проходят**

```bash
npx jest services/__tests__/geocodingApi.test.js
```

Ожидаемый результат: `3 passed`

- [ ] **Step 5: Запустить все тесты**

```bash
npx jest
```

Ожидаемый результат: `7 passed`

- [ ] **Step 6: Закоммитить**

```bash
git add services/geocodingApi.js services/__tests__/geocodingApi.test.js
git commit -m "feat: add geocoding API service with tests"
```

---

## Task 5: WeatherContext — глобальное состояние

**Files:**
- Create: `context/WeatherContext.js`

- [ ] **Step 1: Создать папку**

```bash
mkdir context
```

- [ ] **Step 2: Создать WeatherContext.js**

```javascript
import React, { createContext, useContext, useState } from 'react';
import { fetchCurrentWeather, fetchForecast } from '../services/weatherApi';
import { searchCity } from '../services/geocodingApi';

const WeatherContext = createContext(null);

export function WeatherProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [city, setCity] = useState(null);
  const [coordinates, setCoordinates] = useState(null);
  const [weatherData, setWeatherData] = useState(null);
  const [forecastData, setForecastData] = useState(null);
  const [forecastDays, setForecastDays] = useState(5);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  async function loadWeather(latitude, longitude, cityName, days = forecastDays) {
    setLoading(true);
    setError(null);
    try {
      const [current, forecast] = await Promise.all([
        fetchCurrentWeather(latitude, longitude),
        fetchForecast(latitude, longitude, days),
      ]);
      setWeatherData(current);
      setForecastData(forecast);
      setCoordinates({ latitude, longitude });
      setCity(cityName);
    } catch (e) {
      setError(e.message);
    } finally {
      setLoading(false);
    }
  }

  async function loadByCity(cityName) {
    setLoading(true);
    setError(null);
    try {
      const result = await searchCity(cityName);
      await loadWeather(result.latitude, result.longitude, result.name);
    } catch (e) {
      setError(e.message);
      setLoading(false);
    }
  }

  function toggleTheme() {
    setTheme(prev => (prev === 'light' ? 'dark' : 'light'));
  }

  function changeForecastDays(days) {
    setForecastDays(days);
    if (coordinates) {
      loadWeather(coordinates.latitude, coordinates.longitude, city, days);
    }
  }

  return (
    <WeatherContext.Provider
      value={{
        theme,
        toggleTheme,
        city,
        weatherData,
        forecastData,
        forecastDays,
        changeForecastDays,
        loading,
        error,
        loadWeather,
        loadByCity,
      }}
    >
      {children}
    </WeatherContext.Provider>
  );
}

export function useWeather() {
  const ctx = useContext(WeatherContext);
  if (!ctx) throw new Error('useWeather must be used within WeatherProvider');
  return ctx;
}
```

- [ ] **Step 3: Закоммитить**

```bash
git add context/WeatherContext.js
git commit -m "feat: add WeatherContext with theme, weather state, and actions"
```

---

## Task 6: Навигация в App.js

**Files:**
- Modify: `App.js`

- [ ] **Step 1: Заменить содержимое App.js**

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { WeatherProvider } from './context/WeatherContext';
import HomeScreen from './screens/HomeScreen';
import ForecastScreen from './screens/ForecastScreen';
import SettingsScreen from './screens/SettingsScreen';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <WeatherProvider>
      <NavigationContainer>
        <Stack.Navigator initialRouteName="Home">
          <Stack.Screen name="Home" component={HomeScreen} options={{ title: 'Weather' }} />
          <Stack.Screen name="Forecast" component={ForecastScreen} options={{ title: 'Forecast' }} />
          <Stack.Screen name="Settings" component={SettingsScreen} options={{ title: 'Settings' }} />
        </Stack.Navigator>
      </NavigationContainer>
    </WeatherProvider>
  );
}
```

- [ ] **Step 2: Создать пустые файлы экранов (иначе App.js упадёт с ошибкой импорта)**

```bash
mkdir screens
```

Создай `screens/HomeScreen.js`:
```javascript
import { View, Text } from 'react-native';
export default function HomeScreen() {
  return <View><Text>Home</Text></View>;
}
```

Создай `screens/ForecastScreen.js`:
```javascript
import { View, Text } from 'react-native';
export default function ForecastScreen() {
  return <View><Text>Forecast</Text></View>;
}
```

Создай `screens/SettingsScreen.js`:
```javascript
import { View, Text } from 'react-native';
export default function SettingsScreen() {
  return <View><Text>Settings</Text></View>;
}
```

- [ ] **Step 3: Запустить приложение и проверить навигацию**

```bash
npx expo start
```

Ожидаемый результат: экран с надписью "Home" и заголовком "Weather" в навигационной шапке.

- [ ] **Step 4: Закоммитить**

```bash
git add App.js screens/HomeScreen.js screens/ForecastScreen.js screens/SettingsScreen.js
git commit -m "feat: set up stack navigation with three screens"
```

---

## Task 7: Компонент WeatherIcon

**Files:**
- Create: `components/WeatherIcon.js`

- [ ] **Step 1: Создать папку**

```bash
mkdir components
```

- [ ] **Step 2: Создать WeatherIcon.js**

```javascript
import { Text } from 'react-native';

function codeToEmoji(code) {
  if (code === 0) return '☀️';
  if (code <= 3) return '⛅';
  if (code <= 48) return '🌫️';
  if (code <= 55) return '🌦️';
  if (code <= 65) return '🌧️';
  if (code <= 75) return '❄️';
  if (code <= 82) return '🌧️';
  if (code <= 99) return '⛈️';
  return '🌡️';
}

export function codeToDescription(code) {
  if (code === 0) return 'Clear sky';
  if (code === 1) return 'Mainly clear';
  if (code === 2) return 'Partly cloudy';
  if (code === 3) return 'Overcast';
  if (code <= 48) return 'Foggy';
  if (code <= 55) return 'Drizzle';
  if (code <= 65) return 'Rain';
  if (code <= 75) return 'Snow';
  if (code <= 82) return 'Showers';
  if (code <= 99) return 'Thunderstorm';
  return 'Unknown';
}

export default function WeatherIcon({ code, size = 48 }) {
  return <Text style={{ fontSize: size }}>{codeToEmoji(code)}</Text>;
}
```

- [ ] **Step 3: Закоммитить**

```bash
git add components/WeatherIcon.js
git commit -m "feat: add WeatherIcon component with emoji and description"
```

---

## Task 8: Компонент WeatherCard

**Files:**
- Create: `components/WeatherCard.js`

- [ ] **Step 1: Создать WeatherCard.js**

```javascript
import { View, Text, StyleSheet } from 'react-native';
import WeatherIcon from './WeatherIcon';
import { useWeather } from '../context/WeatherContext';
import { themes } from '../constants/themes';

export default function WeatherCard({ date, maxTemp, minTemp, weathercode }) {
  const { theme } = useWeather();
  const colors = themes[theme];

  const dayName = new Date(date).toLocaleDateString('en-US', { weekday: 'short' });

  return (
    <View style={[styles.card, { backgroundColor: colors.card, borderColor: colors.border }]}>
      <Text style={[styles.day, { color: colors.text }]}>{dayName}</Text>
      <WeatherIcon code={weathercode} size={28} />
      <View style={styles.temps}>
        <Text style={[styles.maxTemp, { color: colors.text }]}>{Math.round(maxTemp)}°</Text>
        <Text style={[styles.minTemp, { color: colors.subtext }]}>{Math.round(minTemp)}°</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: 16,
    marginVertical: 4,
    marginHorizontal: 16,
    borderRadius: 12,
    borderWidth: 1,
  },
  day: { fontSize: 16, width: 50 },
  temps: { flexDirection: 'row', gap: 8 },
  maxTemp: { fontSize: 16, fontWeight: 'bold' },
  minTemp: { fontSize: 16 },
});
```

- [ ] **Step 2: Закоммитить**

```bash
git add components/WeatherCard.js
git commit -m "feat: add WeatherCard component for forecast list"
```

---

## Task 9: HomeScreen

**Files:**
- Modify: `screens/HomeScreen.js`

- [ ] **Step 1: Заменить содержимое HomeScreen.js**

```javascript
import { useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
  ScrollView,
} from 'react-native';
import * as Location from 'expo-location';
import { StatusBar } from 'expo-status-bar';
import { useWeather } from '../context/WeatherContext';
import WeatherIcon, { codeToDescription } from '../components/WeatherIcon';
import { themes } from '../constants/themes';

export default function HomeScreen({ navigation }) {
  const { theme, city, weatherData, loading, error, loadWeather } = useWeather();
  const colors = themes[theme];

  useEffect(() => {
    async function getLocationAndWeather() {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') return;
      const location = await Location.getCurrentPositionAsync({});
      const { latitude, longitude } = location.coords;
      const [place] = await Location.reverseGeocodeAsync({ latitude, longitude });
      const cityName = place.city || place.region || 'Unknown location';
      loadWeather(latitude, longitude, cityName);
    }
    getLocationAndWeather();
  }, []);

  if (loading) {
    return (
      <View style={[styles.center, { backgroundColor: colors.background }]}>
        <ActivityIndicator size="large" color={colors.accent} />
      </View>
    );
  }

  if (error) {
    return (
      <View style={[styles.center, { backgroundColor: colors.background }]}>
        <Text style={[styles.error, { color: colors.text }]}>{error}</Text>
        <TouchableOpacity
          style={[styles.button, { backgroundColor: colors.accent }]}
          onPress={() => navigation.navigate('Settings')}
        >
          <Text style={styles.buttonText}>Enter city manually</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <ScrollView
      style={{ backgroundColor: colors.background }}
      contentContainerStyle={styles.container}
    >
      <StatusBar style={colors.statusBar} />
      {weatherData ? (
        <>
          <Text style={[styles.city, { color: colors.text }]}>{city}</Text>
          <WeatherIcon code={weatherData.weathercode} size={80} />
          <Text style={[styles.temp, { color: colors.text }]}>
            {Math.round(weatherData.temperature_2m)}°C
          </Text>
          <Text style={[styles.description, { color: colors.subtext }]}>
            {codeToDescription(weatherData.weathercode)}
          </Text>
          <View
            style={[
              styles.detailsCard,
              { backgroundColor: colors.card, borderColor: colors.border },
            ]}
          >
            <View style={styles.detail}>
              <Text style={[styles.detailLabel, { color: colors.subtext }]}>Feels like</Text>
              <Text style={[styles.detailValue, { color: colors.text }]}>
                {Math.round(weatherData.apparent_temperature)}°C
              </Text>
            </View>
            <View style={styles.detail}>
              <Text style={[styles.detailLabel, { color: colors.subtext }]}>Humidity</Text>
              <Text style={[styles.detailValue, { color: colors.text }]}>
                {weatherData.relative_humidity_2m}%
              </Text>
            </View>
            <View style={styles.detail}>
              <Text style={[styles.detailLabel, { color: colors.subtext }]}>Wind</Text>
              <Text style={[styles.detailValue, { color: colors.text }]}>
                {Math.round(weatherData.windspeed_10m)} km/h
              </Text>
            </View>
          </View>
          <View style={styles.buttons}>
            <TouchableOpacity
              style={[styles.button, { backgroundColor: colors.accent }]}
              onPress={() => navigation.navigate('Forecast')}
            >
              <Text style={styles.buttonText}>📅 Forecast</Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={[
                styles.button,
                { backgroundColor: colors.card, borderColor: colors.border, borderWidth: 1 },
              ]}
              onPress={() => navigation.navigate('Settings')}
            >
              <Text style={[styles.buttonText, { color: colors.text }]}>⚙️ Settings</Text>
            </TouchableOpacity>
          </View>
        </>
      ) : (
        <Text style={{ color: colors.subtext }}>Determining location...</Text>
      )}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { alignItems: 'center', paddingVertical: 40, paddingHorizontal: 16 },
  center: { flex: 1, alignItems: 'center', justifyContent: 'center', padding: 24 },
  city: { fontSize: 28, fontWeight: 'bold', marginBottom: 8 },
  temp: { fontSize: 64, fontWeight: '200', marginVertical: 8 },
  description: { fontSize: 18, marginBottom: 24 },
  detailsCard: {
    width: '100%',
    flexDirection: 'row',
    justifyContent: 'space-around',
    padding: 20,
    borderRadius: 16,
    borderWidth: 1,
    marginBottom: 32,
  },
  detail: { alignItems: 'center' },
  detailLabel: { fontSize: 13, marginBottom: 4 },
  detailValue: { fontSize: 18, fontWeight: '600' },
  buttons: { width: '100%', gap: 12 },
  button: { paddingVertical: 14, borderRadius: 12, alignItems: 'center' },
  buttonText: { color: '#fff', fontSize: 16, fontWeight: '600' },
  error: { fontSize: 16, textAlign: 'center', marginBottom: 24 },
});
```

- [ ] **Step 2: Запустить приложение и проверить**

```bash
npx expo start
```

Ожидаемый результат: приложение просит разрешение геолокации, затем показывает текущую погоду для твоего местоположения.

- [ ] **Step 3: Закоммитить**

```bash
git add screens/HomeScreen.js
git commit -m "feat: implement HomeScreen with GPS weather and details"
```

---

## Task 10: ForecastScreen

**Files:**
- Modify: `screens/ForecastScreen.js`

- [ ] **Step 1: Заменить содержимое ForecastScreen.js**

```javascript
import { View, Text, StyleSheet, TouchableOpacity, FlatList } from 'react-native';
import { useWeather } from '../context/WeatherContext';
import WeatherCard from '../components/WeatherCard';
import { themes } from '../constants/themes';

const DAY_OPTIONS = [3, 5, 7];

export default function ForecastScreen() {
  const { theme, forecastData, forecastDays, changeForecastDays, city } = useWeather();
  const colors = themes[theme];

  if (!forecastData) {
    return (
      <View style={[styles.center, { backgroundColor: colors.background }]}>
        <Text style={{ color: colors.text }}>No forecast data. Go back and wait for GPS.</Text>
      </View>
    );
  }

  const items = forecastData.time.map((date, i) => ({
    key: date,
    date,
    maxTemp: forecastData.temperature_2m_max[i],
    minTemp: forecastData.temperature_2m_min[i],
    weathercode: forecastData.weathercode[i],
  }));

  return (
    <View style={[styles.container, { backgroundColor: colors.background }]}>
      <Text style={[styles.city, { color: colors.subtext }]}>{city}</Text>
      <View
        style={[
          styles.toggle,
          { backgroundColor: colors.card, borderColor: colors.border },
        ]}
      >
        {DAY_OPTIONS.map(days => (
          <TouchableOpacity
            key={days}
            style={[
              styles.toggleButton,
              forecastDays === days && { backgroundColor: colors.accent },
            ]}
            onPress={() => changeForecastDays(days)}
          >
            <Text
              style={[
                styles.toggleText,
                { color: forecastDays === days ? '#fff' : colors.text },
              ]}
            >
              {days} days
            </Text>
          </TouchableOpacity>
        ))}
      </View>
      <FlatList
        data={items}
        keyExtractor={item => item.key}
        renderItem={({ item }) => (
          <WeatherCard
            date={item.date}
            maxTemp={item.maxTemp}
            minTemp={item.minTemp}
            weathercode={item.weathercode}
          />
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 16 },
  center: { flex: 1, alignItems: 'center', justifyContent: 'center', padding: 24 },
  city: { textAlign: 'center', fontSize: 14, marginBottom: 12 },
  toggle: {
    flexDirection: 'row',
    marginHorizontal: 16,
    marginBottom: 12,
    borderRadius: 10,
    borderWidth: 1,
    overflow: 'hidden',
  },
  toggleButton: { flex: 1, paddingVertical: 10, alignItems: 'center' },
  toggleText: { fontSize: 14, fontWeight: '600' },
});
```

- [ ] **Step 2: Проверить в приложении**

Нажать "Forecast" на главном экране. Ожидаемый результат: список карточек с прогнозом, переключатель 3/5/7 дней работает.

- [ ] **Step 3: Закоммитить**

```bash
git add screens/ForecastScreen.js
git commit -m "feat: implement ForecastScreen with day selector"
```

---

## Task 11: SettingsScreen

**Files:**
- Modify: `screens/SettingsScreen.js`

- [ ] **Step 1: Заменить содержимое SettingsScreen.js**

```javascript
import { useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TextInput,
  TouchableOpacity,
  Switch,
  ActivityIndicator,
} from 'react-native';
import { useWeather } from '../context/WeatherContext';
import { themes } from '../constants/themes';

export default function SettingsScreen({ navigation }) {
  const { theme, toggleTheme, loadByCity, loading, error } = useWeather();
  const colors = themes[theme];
  const [cityInput, setCityInput] = useState('');

  async function handleSearch() {
    if (!cityInput.trim()) return;
    await loadByCity(cityInput.trim());
    navigation.navigate('Home');
  }

  return (
    <View style={[styles.container, { backgroundColor: colors.background }]}>
      <Text style={[styles.label, { color: colors.text }]}>Search city</Text>
      <View style={styles.row}>
        <TextInput
          style={[
            styles.input,
            { backgroundColor: colors.card, color: colors.text, borderColor: colors.border },
          ]}
          placeholder="e.g. London"
          placeholderTextColor={colors.subtext}
          value={cityInput}
          onChangeText={setCityInput}
          onSubmitEditing={handleSearch}
          returnKeyType="search"
        />
        <TouchableOpacity
          style={[styles.searchButton, { backgroundColor: colors.accent }]}
          onPress={handleSearch}
          disabled={loading}
        >
          {loading ? (
            <ActivityIndicator color="#fff" size="small" />
          ) : (
            <Text style={styles.searchButtonText}>Go</Text>
          )}
        </TouchableOpacity>
      </View>
      {error ? (
        <Text style={[styles.errorText, { color: '#e74c3c' }]}>{error}</Text>
      ) : null}
      <View style={[styles.themeRow, { borderTopColor: colors.border }]}>
        <Text style={[styles.label, { color: colors.text }]}>Dark mode</Text>
        <Switch
          value={theme === 'dark'}
          onValueChange={toggleTheme}
          trackColor={{ true: colors.accent }}
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24 },
  label: { fontSize: 16, fontWeight: '600', marginBottom: 8 },
  row: { flexDirection: 'row', gap: 8, marginBottom: 8 },
  input: {
    flex: 1,
    height: 48,
    borderRadius: 10,
    borderWidth: 1,
    paddingHorizontal: 14,
    fontSize: 16,
  },
  searchButton: {
    height: 48,
    paddingHorizontal: 20,
    borderRadius: 10,
    alignItems: 'center',
    justifyContent: 'center',
  },
  searchButtonText: { color: '#fff', fontSize: 16, fontWeight: '600' },
  errorText: { fontSize: 14, marginBottom: 16 },
  themeRow: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingTop: 24,
    marginTop: 16,
    borderTopWidth: 1,
  },
});
```

- [ ] **Step 2: Проверить в приложении**

Перейти в Settings, ввести город (например "Paris"), нажать Go. Ожидаемый результат: возврат на HomeScreen с погодой для Парижа. Переключатель Dark mode меняет тему во всём приложении.

- [ ] **Step 3: Запустить все тесты напоследок**

```bash
npx jest
```

Ожидаемый результат: `7 passed`

- [ ] **Step 4: Финальный коммит**

```bash
git add screens/SettingsScreen.js
git commit -m "feat: implement SettingsScreen with city search and theme toggle"
```
