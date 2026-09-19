# NutriPlus — Procedimiento de release Android privado por APK

## Objetivo

Este documento describe el flujo oficial para generar y actualizar la aplicación Android privada de NutriPlus.

La aplicación no se publica en Google Play. Las versiones se distribuyen mediante APK firmadas por EAS y se instalan directamente en los dispositivos autorizados.

## Identidad de la aplicación

- Nombre: Chatwoot NutriPlus
- Android package: `com.crnutriplus.chatwoot`
- EAS owner: `nutripluscrm`
- EAS project: `chatwoot-mobile`
- EAS project ID: `37e171d9-b9a9-40ac-8bad-5d93d4af8f7a`
- Perfil de release: `production`
- Distribución: `internal`
- Formato Android: `apk`
- Versionado Android: remoto mediante EAS
- `autoIncrement`: habilitado
- Firma Android: keystore remoto administrado por EAS
- Sentry sourcemap upload: deshabilitado actualmente con `SENTRY_DISABLE_AUTO_UPLOAD=true`

## Requisitos previos

Trabajar desde `C:\NP16`.

En PowerShell utilizar `pnpm.cmd`.

Comprobar EAS con:

`pnpm.cmd dlx eas-cli@latest whoami`

Comprobar GitHub CLI con:

`gh auth status`

El repositorio predeterminado de GitHub CLI debe ser:

`crnutriplus/chatwoot-mobile-app`

## Antes de iniciar un release

Ejecutar:

`Set-Location C:\NP16`

`git switch develop`

`git pull --ff-only origin develop`

`git status --short`

`git rev-parse HEAD`

`git status --short` debe quedar vacío.

No generar una build desde una rama que contenga cambios sin commit.

## Verificar configuración de producción

Ejecutar:

`pnpm.cmd dlx eas-cli@latest config --platform android --profile production --non-interactive`

El perfil debe resolver:

- `distribution: internal`
- `environment: production`
- `autoIncrement: true`
- `buildType: apk`

El Android package debe continuar siendo:

`com.crnutriplus.chatwoot`

## Consultar versionCode

Antes de generar una APK:

`pnpm.cmd dlx eas-cli@latest build:version:get --platform android --profile production`

EAS mantiene el `versionCode` remotamente y cada build `production` lo incrementa automáticamente.

## Generar una nueva APK privada

Ejecutar:

`pnpm.cmd dlx eas-cli@latest build --platform android --profile production`

La build debe mantener:

- perfil `production`
- distribución `internal`
- credenciales Android remotas
- package `com.crnutriplus.chatwoot`
- keystore de NutriPlus
- artefacto final `.apk`

No usar `eas submit`.

No subir la aplicación a Google Play.

## Verificar la build

Después de completarse, consultar la build en EAS y confirmar:

- estado `FINISHED`
- plataforma `ANDROID`
- perfil `production`
- `appIdentifier` igual a `com.crnutriplus.chatwoot`
- `appBuildVersion` incrementado
- artefacto APK disponible

## Instalación y actualización

Para que Android permita instalar una nueva APK sobre una versión existente deben mantenerse:

1. El mismo package: `com.crnutriplus.chatwoot`
2. La misma firma Android.
3. Un `versionCode` superior al instalado.

No crear un nuevo keystore para una actualización normal.

No cambiar el Android package.

## Keystore

La configuración actual utiliza:

`Build Credentials WgKXl7wOLY (Default)`

El keystore es JKS y está almacenado remotamente en EAS.

No ejecutar salvo necesidad documentada:

- Set up a new keystore
- Change default keystore
- Delete your keystore

Cambiar o perder la firma impediría actualizar normalmente las instalaciones existentes.

## Firebase

Firebase está configurado para el package:

`com.crnutriplus.chatwoot`

El archivo `google-services.json` se administra mediante la configuración existente de EAS/Firebase.

No publicar su contenido ni copiar secretos al repositorio.

## Pruebas mínimas antes de distribuir

Verificar en un Android real:

- apertura de la app
- inicio de sesión en `https://crm.crnutriplus.com`
- apertura de conversaciones
- envío de mensajes
- acceso al CRM NutriPlus
- navegación CRM → Actions → CRM
- persistencia de sesión
- notificaciones push
- apertura desde una notificación
- comportamiento del WebView del CRM
- actualización sobre la instalación anterior sin desinstalar

## Git y Pull Requests

Todo cambio debe realizarse en una rama separada de `develop`.

GitHub CLI está autenticado como `crnutriplus`.

Usar `gh` para crear y fusionar Pull Requests.

La integración GitHub utilizada desde ChatGPT puede leer y verificar el repositorio, pero actualmente devuelve `403 Resource not accessible by integration` para ciertas operaciones de escritura.

Por eso las operaciones de escritura de Pull Requests se hacen mediante GitHub CLI.

## Después de fusionar un Pull Request

Ejecutar:

`git switch develop`

`git pull --ff-only origin develop`

`git fetch origin --prune`

`git status --short`

`git branch -a`

`git rev-parse HEAD`

Eliminar las ramas de trabajo local y remota después de confirmar el merge.

## Storybook

Expo/EAS puede modificar automáticamente:

`.storybook/storybook.requires.ts`

Si aparece modificado sin ser intencional:

`git restore --source=HEAD --worktree -- .storybook/storybook.requires.ts`

Antes de hacer commit revisar:

`git status --short`

`git diff --check`

## Dependencias

No ejecutar automáticamente:

`expo install --fix`

Las actualizaciones de Expo, React Native y demás dependencias deben tratarse como trabajo separado del release.

## Estado de referencia

Al documentar este procedimiento:

- versión visible de la app: `4.9.3`
- Android `versionCode` remoto confirmado: `2`
- distribución Google Play: no utilizada
- formato oficial de distribución privada: APK
