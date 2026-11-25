# RafeNote & Rafenely.UI.Foundation — Reglas del Repositorio

## Stack y Entorno
- **Runtime:** .NET 10 (`Nullable: enable`, `ImplicitUsings: enable`).
- **App Host:** .NET MAUI Blazor Hybrid (Windows, macOS, Android).
- **Target principal de iteración local:** `net10.0-maccatalyst` (no levantar emuladores de Android salvo que se pida explícitamente).
- **Datos:** EF Core + SQLite (`FileSystem.AppDataDirectory/rafenote.db`).
- **Pruebas:** xUnit, NSubstitute, bUnit, EF Core SQLite `:memory:`.

## Mapa de Proyectos y Fronteras Estrictas (Clean Architecture)
Respeta el grafo de dependencias sin excepciones:

1. **`Rafenely.UI.Foundation`** *(Razor Class Library)*
   - **Propósito:** Librería reutilizable de UI (layout responsivo, navegación DI, toasts, tema, `/settings`).
   - **Prohibido:** Cero referencias a `Microsoft.Maui.*` o `RafeNote.*`.
   - **Contratos clave:** `NavItem`, `IAppFeedbackService`, `IPreferencesStore`, `ISettingSection`, `AddRafenelyUI()`, `RafenelyMainLayout`, `SettingsPage`.
   - **Estilos:** Tokens CSS en `wwwroot/css/rafenely-tokens.css` (modo oscuro por defecto).

2. **`RafeNote.Core`** *(Class Library)*
   - **Propósito:** Dominio puro y casos de uso.
   - **Prohibido:** Cero paquetes NuGet externos (solo BCL de .NET).
   - **Contratos clave:** `Result` / `Result<T>`, `SparkStatus` (`Spark`, `Incubating`, `Archived`), `SparkIdea` (encapsulada con métodos de dominio), `ISparkRepository`, `ISparkEnricher`, `ISparkService`.

3. **`RafeNote.Infrastructure`** *(Class Library)*
   - **Referencias permitidas:** Solo `RafeNote.Core`.
   - **Contratos clave:** `RafeNoteDbContext` (Fluent API, tags serializados a JSON, índices en `Status` y `CreatedAt`), `SparkRepository`, `LocalSparkEnricher`, `AddRafeNoteInfrastructure(string dbPath)`.

4. **`RafeNote.App`** *(.NET MAUI Blazor Hybrid)*
   - **Propósito:** Composition Root y vistas MVP (`Capture.razor` `/`, `Review.razor` `/review`, `Vault.razor` `/vault`).
   - **Regla:** Implementa `MauiPreferencesStore : IPreferencesStore` usando `Preferences.Default`. Usa `RafenelyMainLayout` en `Routes.razor` con `AdditionalAssemblies` apuntando a `Rafenely.UI.Foundation`. No crear un `MainLayout` propio.

5. **`RafeNote.Tests`** *(xUnit)*
   - Cubre lógica de `Core` (mocks con NSubstitute), integración de `Infrastructure` (SQLite `:memory:`) y componentes/DI de `Rafenely.UI.Foundation` (bUnit).

## Estándares de Código
- **SOLID:** Sin lógica de negocio en archivos `.razor`; toda acción delega a `ISparkService` o servicios inyectados.
- **Errores:** Usar `Result` / `Result<T>` para validaciones y flujo de dominio. No lanzar excepciones para control de flujo.
- **EF Core:** Usar `AsNoTracking()` en consultas de solo lectura.
- **Documentación:** Comentarios XML (`/// <summary>`) obligatorios en todas las interfaces públicas y extensiones de `Rafenely.UI.Foundation`.
- **Nulabilidad:** Cero warnings de nulabilidad (CS86xx) al compilar.

## Flujo del Agente (Ahorro de Tokens y Control)
- Ignorar siempre carpetas `bin/`, `obj/`, `.vs/` y `.git/`.
- Ejecutar `dotnet build` y `dotnet test` al concluir cada cambio estructural o fase.
- **Regla de parada:** Si un error de compilación o NuGet se repite 2 veces seguidas, detenerse, reportar el error exacto y esperar instrucciones en lugar de iterar a ciegas.

## Estructura del Workspace (IMPORTANTE)
- El workspace abierto es `RafeNote/`, pero el repositorio Git vive estrictamente en `./RafeNote_App/`.
- Crea `RafeNote.sln` y los 5 proyectos exclusivamente dentro de `RafeNote_App/`.
- Ejecuta todos los comandos `dotnet` y `git` usando `RafeNote_App/` como directorio de trabajo.
- La carpeta `_local/` en la raíz contiene archivos locales fuera de Git (imágenes, data, referencia); nunca crees código fuente ahí.
