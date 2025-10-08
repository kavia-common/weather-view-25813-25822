# Weather App Frontend - Diagrams

## System Context Diagram
The frontend is a single-page React application running in the browser. It communicates with a third-party weather API (e.g., OpenWeather) over HTTPS using an API key provided via CRA environment variables at build time.

```mermaid
flowchart LR
  user["User\n(Web Browser)"]
  app["Weather App Frontend\n(React 18 - CRA)"]
  env["Build-time Env Vars\n(REACT_APP_WEATHER_API_KEY,\nREACT_APP_WEATHER_API_BASE_URL)"]
  api["OpenWeather API\n(HTTPS)"]

  user -->|"HTTP(S) UI"| app
  env -->|"Injected at build time"| app
  app -->|"GET current weather\nwith API key"| api
  api -->|"JSON response"| app
```

## Component Hierarchy
Initial implementation has a single App component with theming. Planned components are shown for weather functionality, keeping state local to App and using hooks.

```mermaid
graph TD
  A["App (root)\n- theme state\n- (planned) cityQuery, weatherData, loading, error"] 
  A --> B["ThemeToggle (planned)\n- toggles data-theme"]
  A --> C["SearchBar (planned)\n- input & submit"]
  A --> D["WeatherCard (planned)\n- shows temperature, condition, humidity, wind"]
  A --> E["LoadingSpinner (planned)"]
  A --> F["ErrorMessage (planned)"]
```

## Data Flow for Weather Lookup
This shows the one-way data flow from user input to the rendered view, with environment variables used by the service at build time.

```mermaid
flowchart TB
  input["User enters city\n(SearchBar)"]
  submit["User submits\n(Search action)"]
  appState["App State\n(cityQuery, loading=true, error=null)"]
  svc["weatherApi.getCurrentWeather(city)\n- uses REACT_APP_WEATHER_API_BASE_URL\n- uses REACT_APP_WEATHER_API_KEY"]
  api["OpenWeather API"]
  resp["Normalize JSON response\n{ city, temperatureC, condition, humidityPct, windKph, iconUrl }"]
  success["Update App State\nloading=false\nweatherData=normalized"]
  failure["Update App State\nloading=false\nerror=message"]
  renderLoading["Render LoadingSpinner"]
  renderError["Render ErrorMessage"]
  renderCard["Render WeatherCard"]

  input --> submit --> appState --> svc --> api
  api --> resp --> success --> renderCard
  api --> failure --> renderError
  appState --> renderLoading
```

## Sequence Diagram for User Search
A sequence showing user interaction, component behavior, and the external API call.

```mermaid
sequenceDiagram
  participant U as User
  participant SB as SearchBar
  participant A as App
  participant WA as weatherApi
  participant OW as OpenWeather API

  U->>SB: Type city name
  U->>SB: Submit search
  SB->>A: onSubmit(city)
  A->>A: set loading=true, error=null
  A->>WA: getCurrentWeather(city)
  WA->>OW: GET /weather?query=city&appid=API_KEY
  OW-->>WA: 200 OK (JSON)
  WA-->>A: Normalized weather data
  A->>A: set loading=false, weatherData=data
  A-->>U: Render WeatherCard

  rect rgba(255,0,0,0.05)
    Note over OW,WA: Error path
    OW-->>WA: 4xx/5xx error
    WA-->>A: Throw/Error result
    A->>A: set loading=false, error=message
    A-->>U: Render ErrorMessage
  end
```

## Deployment Diagram (Development)
Local development setup using CRA with react-scripts, environment variables loaded via .env.local.

```mermaid
graph LR
  dev["Developer Machine"]
  node["Node.js + react-scripts\n(npm start / build / test)"]
  env[".env.local\nREACT_APP_WEATHER_API_KEY\nREACT_APP_WEATHER_API_BASE_URL"]
  browser["Browser (localhost:3000)\nSPA (React 18)"]
  api["OpenWeather API\n(Internet)"]

  dev --> node
  env --> node
  node -->|"Serves bundled assets"| browser
  browser -->|"Fetch weather data (HTTPS)"| api
```

## Notes
- The application remains a single-page CRA app without routing.
- Environment variables must be prefixed with REACT_APP_ to be available at build time.
- All network calls are performed client-side from the browser to the OpenWeather API over HTTPS.
