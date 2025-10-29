
# Instrucciones para Mensajes de Commit

Genera mensajes de commit en español siguiendo estas reglas estrictas:

## Formato
```
<tipo>(<scope>): <descripción>

[cuerpo opcional]

[footer opcional]
```

## Tipos (obligatorio)
- **feat**: Nueva funcionalidad
- **fix**: Corrección de bug
- **docs**: Solo cambios en documentación
- **style**: Cambios de formato (espacios, comas, etc. sin afectar código)
- **refactor**: Refactorización de código (ni fix ni feat)
- **perf**: Mejoras de rendimiento
- **test**: Agregar o corregir tests
- **build**: Cambios en el sistema de build o dependencias
- **ci**: Cambios en configuración de CI
- **chore**: Tareas de mantenimiento (sin cambios en src o test)
- **revert**: Revertir un commit anterior

## Reglas de la descripción
1. Máximo 50 caracteres
2. Imperativo presente: "agrega" NO "agregado" ni "agregando"
3. Sin punto final
4. Minúsculas después de los dos puntos
5. Ser específico y conciso

## Scope (opcional pero recomendado)
- Componente, módulo o área afectada: `auth`, `api`, `ui`, `db`, etc.

## Cuerpo (opcional)
- Separar con línea en blanco
- Explicar el QUÉ y POR QUÉ, no el CÓMO
- Máximo 72 caracteres por línea
- Usar viñetas con `-` o `*` si es necesario

## Footer (opcional)
- `BREAKING CHANGE:` para cambios que rompen compatibilidad
- Referencias a issues: `Closes #123` o `Refs #456`

## Ejemplos
```
feat(auth): agrega autenticación con Google OAuth

Implementa el flujo completo de OAuth 2.0 para permitir login
con cuentas de Google. Incluye manejo de tokens y refresh.

Closes #234
```

```
fix(api): corrige timeout en endpoint de usuarios

El timeout ocurría cuando había más de 1000 usuarios.
Se optimiza la query agregando índice en created_at.
```

```
refactor(utils): simplifica función de validación de emails
```

```
docs(readme): actualiza instrucciones de instalación
```

## Errores comunes a evitar
❌ "actualizaciones varias"
❌ "fix"
❌ "cambios en código"
❌ "WIP"
✅ "feat(cart): agrega botón de compra rápida"
✅ "fix(login): corrige redirección después del logout"