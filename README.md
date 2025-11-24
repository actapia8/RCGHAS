# RCGHAS - GitHub Advanced Security Configuration

Este repositorio contiene la configuración completa de GitHub Advanced Security (GHAS) para análisis de seguridad automatizado.

## 📋 Tabla de Contenidos

- [Workflows Configurados](#workflows-configurados)
- [Requisitos Previos](#requisitos-previos)
- [Configuración Detallada](#configuración-detallada)
- [Flujo de Trabajo](#flujo-de-trabajo)
- [Troubleshooting](#troubleshooting)

---

## 🔧 Workflows Configurados

### 1. CodeQL Daily Scan (`.github/workflows/codeql-daily.yml`)

**Propósito**: Análisis estático de código (SAST) para detectar vulnerabilidades de seguridad.

**Configuración**:
- **Disparador**: Ejecución programada diariamente a las 12:00 PM UTC
- **Rama escaneada**: `master`
- **Lenguaje configurado**: `javascript`
- **Ejecución manual**: Disponible vía `workflow_dispatch`

**¿Cómo funciona?**
1. Checkout de la rama `master`
2. Inicializa CodeQL con el lenguaje especificado
3. Autobuild del proyecto
4. Análisis de seguridad y generación de alertas

**Modificar el lenguaje**:
```yaml
matrix:
  language: [ 'javascript' ] # Cambiar a: python, go, java, etc.
```

---

### 2. Dependency Review (`.github/workflows/dependency-review.yml`)

**Propósito**: Revisar vulnerabilidades en dependencias cuando se crean Pull Requests.

**Configuración**:
- **Disparador**: Pull Requests hacia las ramas `dev`, `cert`, `prd`
- **Tipos de eventos**: `opened`, `synchronize`, `reopened`
- **Severidad mínima**: `moderate`
- **Permisos**: `contents: read`, `pull-requests: write`

**Características avanzadas**:

| Característica | Configuración | Descripción |
|----------------|---------------|-------------|
| **Severidad** | `moderate` | Falla el check si encuentra vulnerabilidades de severidad moderate, high o critical |
| **Scopes** | `runtime, development` | Revisa dependencias de producción y desarrollo |
| **Licencias permitidas** | MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, 0BSD | Solo permite estas licencias open source |
| **Licencias prohibidas** | GPL-2.0, GPL-3.0, AGPL-3.0, LGPL-* | Bloquea licencias con problemas legales |
| **Comentarios en PR** | `always` | Comenta automáticamente en el PR con el resumen de seguridad |
| **OpenSSF Scorecard** | `true` | Muestra el score de seguridad (0-10) de cada dependencia |
| **Verificación de vulnerabilidades** | `true` | Valida contra bases de datos de vulnerabilidades conocidas |

**Ejemplo de flujo**:
```
feature/nueva-funcionalidad → dev → Dependency Review se ejecuta automáticamente
```

**⚠️ Importante**:
- El workflow **NO se ejecuta** en PRs hacia `master`
- Solo se activa en PRs hacia: `dev`, `cert`, `prd`

---

### 3. Dependabot (Deshabilitado)

**Estado**: ❌ Eliminado del repositorio

**Razón**: Se eliminó la configuración de Dependabot para **evitar Pull Requests automáticos**. Todos los PRs deben ser creados manualmente.

Si se requiere en el futuro, crear `.github/dependabot.yml`:
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    target-branch: "feature"
```

---

## ✅ Requisitos Previos

### 1. Repositorio Público o GitHub Enterprise

- **Repositorio público**: GHAS es **gratuito**
- **Repositorio privado**: Requiere licencia de **GitHub Enterprise** con GHAS habilitado

### 2. Habilitar GitHub Advanced Security

Ve a: **Settings** → **Security** → **Code security and analysis**

Habilita las siguientes opciones:

| Opción | Estado Requerido | Descripción |
|--------|------------------|-------------|
| **Dependency graph** | ✅ Enabled | Necesario para Dependency Review |
| **Dependabot alerts** | ✅ Enabled | Alertas de vulnerabilidades en dependencias |
| **Code scanning** | ✅ Enabled | Para CodeQL analysis |
| **Secret scanning** | ⚠️ Opcional | Detecta secretos en el código |

### 3. Estructura de Ramas

El repositorio debe tener las siguientes ramas:

```
master (producción)
  ↑
dev (desarrollo)
  ↑
cert (certificación)
  ↑
prd (pre-producción)
  ↑
feature/* (ramas de funcionalidades)
```

---

## 🔄 Flujo de Trabajo

### Escenario 1: Análisis Diario Automático

```
12:00 PM UTC → CodeQL escanea rama master → Genera alertas si encuentra vulnerabilidades
```

**Resultado**: Las vulnerabilidades aparecen en **Security** → **Code scanning alerts**

---

### Escenario 2: Pull Request Manual

```
1. Desarrollador crea PR: feature/nueva-funcionalidad → dev
2. GitHub ejecuta automáticamente: Dependency Review
3. El workflow verifica:
   ✓ Vulnerabilidades en dependencias nuevas/modificadas
   ✓ Licencias de las dependencias
   ✓ OpenSSF Scorecard de los paquetes
4. Resultados:
   ✅ Check pasa → El PR puede fusionarse
   ❌ Check falla → Bloquea el merge (requiere corrección)
5. El bot comenta en el PR con el resumen detallado
```

**Ejemplo de comentario automático**:
```
🔒 Dependency Review Summary

Vulnerabilities Found: 2
- express@4.17.1: High severity CVE-2022-XXXXX
- lodash@4.17.19: Moderate severity CVE-2021-XXXXX

License Issues: 1
- package-foo: Uses GPL-3.0 (prohibited)

OpenSSF Scorecard:
- express: 7.2/10
- lodash: 6.8/10
```

---

### Escenario 3: PR hacia Master (NO ejecuta Dependency Review)

```
dev → master: Dependency Review NO se ejecuta
```

**Razón**: Por configuración, solo se ejecuta en PRs hacia `dev`, `cert`, `prd`.

**Para habilitar también en master**, editar `.github/workflows/dependency-review.yml`:
```yaml
on:
  pull_request:
    branches: [ "master", "dev", "cert", "prd" ]  # Agregar master
```

---

## 🐛 Troubleshooting

### Error: "Dependency review is not supported on this repository"

**Causa**: Dependency Graph no está habilitado.

**Solución**:
1. Ve a **Settings** → **Security** → **Code security and analysis**
2. Busca **Dependency graph**
3. Haz clic en **Enable**
4. Espera 5-10 minutos para que se procese
5. Re-ejecuta el workflow del PR

---

### Error: "Resource not accessible by personal access token"

**Causa**: El token de GitHub no tiene permisos suficientes.

**Solución**:
1. Ve a **Settings** → **Actions** → **General**
2. En **Workflow permissions**, selecciona:
   - ✅ **Read and write permissions**
   - ✅ **Allow GitHub Actions to create and approve pull requests**
3. Guarda los cambios

---

### El workflow no se ejecuta en mi PR

**Verifica**:

1. **Rama destino**: El PR debe ir hacia `dev`, `cert`, o `prd`
   - ❌ `feature/x → master` NO ejecuta Dependency Review
   - ✅ `feature/x → dev` SÍ ejecuta Dependency Review

2. **Eventos**: El workflow se activa en:
   - `opened` (PR creado)
   - `synchronize` (nuevo commit al PR)
   - `reopened` (PR reabierto)

3. **Permisos**: Verifica que Actions esté habilitado:
   - **Settings** → **Actions** → **General**
   - Debe estar en **Allow all actions**

---

### CodeQL no encuentra mi lenguaje

**Solución**: Editar `.github/workflows/codeql-daily.yml`:

```yaml
matrix:
  language: [ 'javascript', 'python', 'go' ]  # Agregar múltiples lenguajes
```

Lenguajes soportados: `javascript`, `typescript`, `python`, `java`, `csharp`, `cpp`, `go`, `ruby`

---

## 📊 Monitoreo y Reportes

### Ver Alertas de Seguridad

1. **Code Scanning Alerts**: **Security** → **Code scanning**
2. **Dependency Alerts**: **Security** → **Dependabot alerts**
3. **Workflow Runs**: **Actions** → Ver ejecuciones de workflows

### Métricas Clave

- **CodeQL Daily Scan**: Revisa la pestaña **Actions** → **CodeQL Daily Scan**
- **Dependency Review**: Revisa los checks en cada PR
- **Tiempo de ejecución**: ~1-2 minutos para Dependency Review, ~5-10 minutos para CodeQL

---

## 🔐 Mejores Prácticas

1. ✅ **No deshabilitar los checks**: Si un check falla, corrige el problema en lugar de forzar el merge
2. ✅ **Revisar los comentarios del bot**: Contienen información detallada sobre vulnerabilidades
3. ✅ **Actualizar dependencias regularmente**: Evita acumulación de vulnerabilidades
4. ✅ **Usar ramas protegidas**: Configura branch protection rules para requerir checks pasados
5. ✅ **Monitorear alertas**: Revisa semanalmente la pestaña Security

---

## 📝 Notas Adicionales

- **Repositorio actual**: Público (GHAS gratuito)
- **PRs automáticos**: Deshabilitados (sin Dependabot)
- **Modo de operación**: Solo PRs manuales
- **Ramas protegidas por Dependency Review**: `dev`, `cert`, `prd`
- **Rama analizada por CodeQL**: `master`

---

## 🤝 Contribuir

Para agregar nuevas funcionalidades:
1. Crear rama desde `dev`: `git checkout -b feature/nueva-funcionalidad`
2. Hacer cambios y commit
3. Crear PR hacia `dev`
4. Esperar que Dependency Review pase
5. Solicitar revisión y merge

---

## 📚 Referencias

- [GitHub Advanced Security Documentation](https://docs.github.com/en/code-security)
- [CodeQL Documentation](https://codeql.github.com/docs/)
- [Dependency Review Action](https://github.com/actions/dependency-review-action)
- [OpenSSF Scorecard](https://github.com/ossf/scorecard)
