# Weather App Frontend - Architecture and Implementation Guide

## Overview
This document provides an architecture overview, implementation plan, coding standards, testing strategy, and future enhancements for the Weather App Frontend. The current codebase is a Create React App (CRA) React 18 template with:
- A single App component (src/App.js) that includes a theme toggle using CSS variables and a dark/light attribute on the document element.
- CRA tooling (react-scripts) for build, test, and start.
- Basic unit test (src/App.test.js).
- ESLint configuration (eslint.config.mjs) and standard CRA linting via package.json.

Refer to the diagrams for visual context:
- See docs/DIAGRAMS.md for System Context, Component Hierarchy, Data Flow, Sequence, and Deployment diagrams.

## Tech Stack
- Framework: React 18 (CRA)
- Tooling: react-scripts (start/build/test)
- Styling: CSS with custom properties (variables) and basic responsive rules in src/App.css
- Testing: Jest with React Testing Library (via CRA), setup in src/setupTests.js
- Linting: ESLint with eslint-plugin-react and CRA defaults
- Bundling: Webpack (managed by CRA)
- No routing, no state library, no API integration implemented yet

## Application Structure
- src/index.js: Bootstraps the React application, enabling StrictMode and rendering App.
- src/App.js: Main component managing light/dark theme toggle and base layout.
- src/App.css: Defines light/dark theme variables and component styles.
- src/index.css: Global typography and base CSS rules.
- src/logo.svg: React logo asset.
- src/App.test.js and src/setupTests.js: Unit test and Jest setup.

A future structure for weather features will extend src with components and utilities:
- src/components/
  - SearchBar.jsx
  - WeatherCard.jsx
  - ThemeToggle.jsx (optional extraction from App)
  - LoadingSpinner.jsx
  - ErrorMessage.jsx
- src/services/
  - weatherApi.js
- src/hooks/
  - useWeather.js (optional)
- src/styles/
  - variables.css (optional centralization of theme variables)

## Components (Planned)
- App: Root component. Manages theme state, provides structure and layout.
- SearchBar: Controlled input with submit to trigger fetch for city weather.
- WeatherCard: Displays current temperature, condition/description, humidity, wind, and city name.
- LoadingSpinner: Displayed while fetching.
- ErrorMessage: Displayed on fetch or input errors.
- ThemeToggle: Button to switch between light/dark modes; may remain in App initially.

## State Management
- Local component state via useState/useEffect is sufficient for initial scope.
- App will maintain:
  - theme: 'light' | 'dark'
  - cityQuery: string (from SearchBar)
  - weatherData: object | null
  - loading: boolean
  - error: string | null
- Optional: Extract data fetching logic to a custom hook (useWeather) for reusability and testability.

## Theming
- Continue to use CSS custom properties defined in src/App.css and toggle via data-theme attribute on documentElement.
- Ensure new components use variables instead of hardcoded colors.
- Follow the Ocean Professional style guidelines: modern aesthetic, rounded corners, subtle shadows, gradients.

## Build, Test, and Lint Tooling
- Build: npm run build (react-scripts build)
- Start: npm start (react-scripts start)
- Test: npm test (react-scripts test)
- Lint: eslint configured via eslint.config.mjs and CRA defaults
  - Current rules include @eslint/js recommended and eslint-plugin-react; react-in-jsx-scope is off to reflect React 17+ JSX transform.
- Formatting: Recommend adding Prettier and a formatting script; not yet present in the codebase.

## Implementation Plan (Initial Weather Features)

### 1) Components and Structure
- Create src/components/SearchBar.jsx
  - Props: value, onChange, onSubmit, placeholder
  - Renders input + submit button; handles Enter key
- Create src/components/WeatherCard.jsx
  - Props: city, temperature, condition, humidity, wind
  - Renders a card with responsive styling using CSS variables
- Create src/components/LoadingSpinner.jsx
  - Simple spinner using CSS animations
- Create src/components/ErrorMessage.jsx
  - Props: message
  - Styled alert/notice using theme variables
- Optionally extract the theme button to src/components/ThemeToggle.jsx

Update App.js to:
- Hold state: cityQuery, weatherData, loading, error, theme
- Wire SearchBar submit to trigger fetch (weatherApi.getCurrentWeather(cityQuery))
- Render LoadingSpinner or ErrorMessage as needed
- Render WeatherCard when weatherData is available

### 2) API Integration
- Create src/services/weatherApi.js
  - Export async function getCurrentWeather(city)
  - Read base URL and API key from environment variables
  - Implement fetch with proper query params and error handling
  - Normalize response (map external fields to a simple shape):
    - { city, temperatureC, condition, humidityPct, windKph, iconUrl }
- Show loading state while request is in flight; display error message on failure.

### 3) Environment Variables
- Create .env.local (not committed) with:
  - REACT_APP_WEATHER_API_BASE_URL=https://api.openweathermap.org/data/2.5
  - REACT_APP_WEATHER_API_KEY=your_api_key
- Document usage in README and this doc. CRA exposes variables prefixed with REACT_APP_.

### 4) Styling and Accessibility
- Extend App.css with classes for weather card, inputs, buttons, and error messages.
- Ensure proper aria-labels and roles for interactive elements.
- Maintain adequate color contrast for both light and dark themes.

### 5) Testing
- Unit tests:
  - SearchBar: input change, submit callback fired
  - WeatherCard: renders values correctly
  - App: theme toggle updates aria-label and text
- Integration (mocked):
  - Happy path: entering a city triggers fetch and displays weather
  - Error path: API error displays ErrorMessage
  - Loading path: displays LoadingSpinner while waiting

## Coding Standards
- JavaScript/React
  - Use functional components and hooks
  - Prefer explicit prop names and default values where sensible
  - Keep components focused; extract helpers/utilities for formatting values (e.g., temperature)
- ESLint
  - Keep eslint.config.mjs current; include no-unused-vars and react/jsx-uses-vars
  - Consider enabling rules around hooks (react-hooks/rules-of-hooks, react-hooks/exhaustive-deps)
- Prettier
  - Add Prettier to devDependencies and a config file (.prettierrc) for consistent formatting
  - Scripts:
    - "format": "prettier --write ."
    - "lint": "eslint ."
- Naming and Structure
  - Use PascalCase for components, camelCase for variables and functions
  - Co-locate tests as ComponentName.test.jsx or in __tests__ folders

## Testing Strategy
- Unit tests with Jest and React Testing Library
- Mock fetch in tests for API interactions
- Aim for coverage on:
  - Controlled inputs and event handling (SearchBar)
  - Conditional rendering and state transitions (App)
  - Data display formatting (WeatherCard)
- CI-friendly: npm test (CI=true if needed) with clear pass/fail

## Future Enhancements
- Forecasts: Hourly and 7-day views with tabs or toggle
- Geolocation: Option to use browser geolocation for “current location” weather
- Favorites: Save and quickly access favorite cities (localStorage)
- Internationalization: Support for multiple languages and units (C/F, km/h/mph)
- Caching: Basic in-memory caching to reduce repeated API calls
- Accessibility: More comprehensive ARIA and keyboard navigation support
- Theming: Theme persistence via localStorage and more granular theme tokens
- Performance: Code-splitting of heavy components, image/icon optimization
- Routing: Introduce react-router if navigation becomes necessary

## References
- Current Codebase Files Referenced:
  - src/App.js
  - src/App.css
  - src/index.js
  - src/index.css
  - src/App.test.js
  - src/setupTests.js
  - package.json
  - eslint.config.mjs

