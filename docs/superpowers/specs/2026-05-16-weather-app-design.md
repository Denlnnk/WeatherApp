# Weather App — Design Spec
Date: 2026-05-16

## Overview

Учебное мобильное приложение погоды на Expo + React Native. Цель — научиться создавать мобильные приложения: навигация, API-запросы, глобальное состояние, переключение тем.

## Экраны

### HomeScreen (главный)
- Определение местоположения по GPS при запуске
- Отображение: город, температура, описание погоды, иконка, ощущаемая температура, влажность, скорость ветра
- Кнопка перехода на ForecastScreen

### ForecastScreen (прогноз)
- Список дней с температурой min/max и иконкой погоды
- Переключатель количества дней: 3 / 5 / 7
- Кнопка "назад" на HomeScreen

### SettingsScreen (настройки)
- Поле ввода города вручную (переопределяет GPS)
- Переключатель темы: тёмная / светлая
- Доступен с HomeScreen

## Архитектура

### Навигация
Библиотека `react-navigation` (stack navigator). Переходы: Home → Forecast, Home → Settings.

### Данные (State)
React Context (`WeatherContext`) хранит глобальное состояние:
- `weatherData` — текущая погода
- `forecastData` — прогноз
- `city` — выбранный город
- `theme` — текущая тема (`light` | `dark`)
- `forecastDays` — количество дней прогноза (3 / 5 / 7)

### API
**Open-Meteo** (бесплатно, без регистрации, без API-ключа).
- Геолокация: `expo-location` для GPS-координат
- Запрос текущей погоды: `https://api.open-meteo.com/v1/forecast`
- Запрос прогноза: тот же endpoint с параметром `daily`

### Структура файлов
```
WeatherApp/
├── App.js                      ← точка входа, настройка навигации
├── screens/
│   ├── HomeScreen.js
│   ├── ForecastScreen.js
│   └── SettingsScreen.js
├── components/
│   ├── WeatherIcon.js
│   └── WeatherCard.js
├── services/
│   └── weatherApi.js
├── context/
│   └── WeatherContext.js
└── assets/
```

## Обработка ошибок
- GPS недоступен → показать сообщение, предложить ввести город вручную
- API недоступен → показать сообщение об ошибке на экране
- Загрузка данных → показать индикатор загрузки (ActivityIndicator)

## Тема
Два набора цветов (светлый/тёмный) определены в `WeatherContext`. Все экраны читают тему из контекста и применяют соответствующие стили.

## Зависимости (библиотеки)
- `@react-navigation/native` — навигация
- `@react-navigation/stack` — stack navigator
- `expo-location` — доступ к GPS
- `react-native-safe-area-context` — корректные отступы на разных телефонах
- `react-native-screens` — оптимизация навигации
