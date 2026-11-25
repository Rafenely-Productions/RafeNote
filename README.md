# RafeNote

Captura de ideas sin fricción antes de que se evaporen. Por ahora solo el esqueleto para atrapar chispazos en menos de 3 segundos; el resto viene en camino.

## Arquitectura

Construido sobre **.NET 10 MAUI Blazor Hybrid**, **Clean Architecture** y **SQLite**, sirviendo también como campo de pruebas para la librería reutilizable **`Rafenely.UI.Foundation`**.

- **`Rafenely.UI.Foundation`** — Razor Class Library desacoplada (layout responsivo, navegación por DI, toasts y `/settings`).
- **`RafeNote.Core`** — Dominio puro, casos de uso y contratos (cero dependencias externas).
- **`RafeNote.Infrastructure`** — EF Core + SQLite y servicios de enriquecimiento.
- **`RafeNote.App`** — Host multiplataforma (macOS, Windows, Android).
- **`RafeNote.Tests`** — Pruebas unitarias y de integración (`xUnit`, `NSubstitute`, `bUnit`).

## Estado actual

En construcción (MVP inicial).
