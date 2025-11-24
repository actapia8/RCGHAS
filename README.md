# RCGHAS - GitHub Advanced Security Pipeline

Este repositorio contiene la configuración de pipelines de seguridad utilizando GitHub Advanced Security.

## Workflows Configurados

### 1. Dependency Review (`.github/workflows/dependency-review.yml`)
- **Propósito**: Escanear dependencias en busca de vulnerabilidades conocidas.
- **Disparador**: Pull Requests hacia las ramas `dev`, `cert`, y `prd`.
- **Acción**: Bloquea el PR si se encuentran vulnerabilidades de severidad Alta o Crítica.

### 2. CodeQL Daily Scan (`.github/workflows/codeql-daily.yml`)
- **Propósito**: Análisis estático de código (SAST) para encontrar vulnerabilidades de seguridad.
- **Disparador**: Ejecución programada todos los días a las 12:00 PM (UTC) en la rama `master`.
- **Lenguaje Configurado**: Javascript (Por defecto). *Editar el archivo workflow para cambiar el lenguaje si es necesario.*

### 3. Dependabot (`.github/dependabot.yml`)
- **Propósito**: Mantener las dependencias actualizadas automáticamente.
- **Configuración**: Revisa actualizaciones diariamente para `npm` y `github-actions`.

## Requisitos Previos

- El repositorio debe tener **GitHub Advanced Security** habilitado en la configuración (Settings > Security & analysis).
- Para repositorios privados, esto requiere una licencia de GHAS. Para repositorios públicos, es gratuito.
Ultimo cambio

Prueba 1
