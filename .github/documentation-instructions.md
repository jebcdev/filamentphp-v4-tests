# Instrucciones para Documentación de Código

Documenta TODO el código en español siguiendo estos estándares profesionales:

## Docstrings de Funciones/Métodos

Usa este formato (adaptar según lenguaje):

```python
def nombre_funcion(param1: tipo, param2: tipo) -> tipo_retorno:
    """
    Breve descripción de una línea de lo que hace la función.
    
    Descripción detallada del propósito y comportamiento de la función.
    Explica casos de uso, algoritmos importantes o lógica compleja.
    
    Args:
        param1: Descripción del parámetro, su propósito y valores esperados.
               Si tiene restricciones o formatos específicos, mencionarlos.
        param2: Descripción del segundo parámetro.
    
    Returns:
        Descripción detallada de qué retorna la función.
        Incluye el tipo y estructura del dato retornado.
    
    Raises:
        TipoError: Cuándo y por qué se lanza esta excepción.
        OtroError: Descripción de otra posible excepción.
    
    Example:
        >>> resultado = nombre_funcion("valor1", 123)
        >>> print(resultado)
        'salida esperada'
    
    Note:
        Información adicional importante, limitaciones o consideraciones
        de rendimiento. Complejidad temporal: O(n).
    
    Warning:
        Advertencias sobre uso incorrecto o casos edge.
    
    See Also:
        otra_funcion_relacionada: Descripción breve de la relación.
    
    References:
        - RFC o especificación seguida
        - Artículo o documentación relevante
    """
```

## Docstrings de Clases

```python
class NombreClase:
    """
    Descripción breve de la responsabilidad de la clase.
    
    Descripción detallada del propósito de la clase, su rol en el sistema
    y patrones de diseño implementados si aplica.
    
    Attributes:
        atributo1: Descripción del atributo y su propósito.
        atributo2: Otro atributo con su tipo y uso.
    
    Example:
        >>> instancia = NombreClase(param1, param2)
        >>> instancia.metodo()
        'resultado'
    
    Note:
        Thread-safety, inmutabilidad u otras características importantes.
    """
```

## Comentarios Inline (para cada línea compleja)

### Reglas:
1. **Comenta el POR QUÉ, no el QUÉ** (el código ya dice qué hace)
2. Usa `#` con un espacio después
3. Alinea comentarios verticalmente cuando sea posible
4. Máximo 72-79 caracteres por línea

### Cuándo comentar cada línea:
- ✅ Lógica de negocio no obvia
- ✅ Algoritmos complejos
- ✅ Workarounds o hacks necesarios
- ✅ Números mágicos o constantes
- ✅ Regex complejas
- ✅ Optimizaciones no evidentes
- ❌ Código autoexplicativo

### Formato de comentarios inline:

```python
# === SECCIÓN: Procesamiento de datos ===
# Iteramos en reversa para evitar problemas con índices al eliminar elementos
for i in range(len(lista) - 1, -1, -1):
    # Validamos integridad antes de procesar para prevenir datos corruptos
    if not validar_integridad(lista[i]):
        # Logging crítico: este caso indica fallo en la capa de validación previa
        logger.error(f"Dato corrupto en índice {i}")
        del lista[i]  # Eliminación segura iterando en reversa
        continue
    
    # Aplicamos transformación según spec RFC-1234 sección 3.2
    resultado = transformar(lista[i])
    
    # Cache del resultado para evitar recálculos (mejora 40% rendimiento)
    cache[lista[i].id] = resultado

# HACK: Timeout extendido a 60s porque el API externa es lenta
# TODO: Migrar a cola asíncrona en próxima versión (ticket #789)
response = requests.get(url, timeout=60)

# FIXME: Este manejo de errores es temporal hasta implementar retry logic
if response.status_code != 200:
    return None  # Retornamos None para mantener compatibilidad con v1.x
```

## Comentarios de Sección

Para dividir archivos largos:

```python
# ==============================================================================
# CONSTANTES Y CONFIGURACIÓN
# ==============================================================================

MAX_RETRIES = 3  # Número máximo de reintentos antes de fallar
TIMEOUT = 30     # Timeout en segundos para requests HTTP


# ==============================================================================
# FUNCIONES AUXILIARES
# ==============================================================================

def helper_function():
    pass


# ==============================================================================
# CLASE PRINCIPAL
# ==============================================================================

class MiClase:
    pass
```

## Comentarios Especiales

```python
# TODO: Descripción de tarea pendiente (asignar a: @usuario, prioridad: alta)
# FIXME: Descripción del bug que necesita corrección urgente
# HACK: Explicación de por qué se usa esta solución temporal
# NOTE: Información importante para entender el contexto
# WARNING: Advertencia sobre uso peligroso o efectos secundarios
# OPTIMIZE: Oportunidad de optimización identificada
# DEPRECATED: Esta función será removida en v2.0, usar nueva_funcion() en su lugar
```

## Docstrings de Módulos

Al inicio de cada archivo:

```python
"""
Módulo para gestión de autenticación y autorización.

Este módulo implementa el sistema de autenticación JWT siguiendo
el estándar RFC 7519. Provee funciones para generar, validar y
refrescar tokens de acceso.

Exports:
    - generar_token: Crea un nuevo JWT
    - validar_token: Verifica la validez de un JWT
    - RefreshToken: Clase para manejar tokens de refresco

Dependencies:
    - PyJWT >= 2.0.0
    - cryptography >= 3.4.0

Author: Tu Nombre
Created: 2024-03-15
Last Modified: 2024-03-20

Examples:
    Uso básico del módulo:
    
    >>> from auth import generar_token, validar_token
    >>> token = generar_token(user_id=123)
    >>> validar_token(token)
    True
"""
```

## Para APIs REST (endpoints):

```python
@app.post("/api/usuarios")
async def crear_usuario(usuario: UsuarioCreate):
    """
    Crea un nuevo usuario en el sistema.
    
    Endpoint: POST /api/usuarios
    Auth: Requiere token JWT con rol 'admin'
    Rate limit: 10 requests/minuto
    
    Args:
        usuario: Objeto con datos del usuario a crear
            - email (str): Email único del usuario
            - nombre (str): Nombre completo (3-100 caracteres)
            - edad (int, opcional): Edad del usuario
    
    Returns:
        UsuarioResponse: Usuario creado con su ID generado
        Status: 201 Created
    
    Raises:
        HTTPException 400: Email ya existe o datos inválidos
        HTTPException 401: Token inválido o expirado
        HTTPException 403: Usuario sin permisos de admin
        HTTPException 422: Validación de datos falló
        HTTPException 500: Error interno del servidor
    
    Example Request:
        ```json
        {
            "email": "usuario@example.com",
            "nombre": "Juan Pérez",
            "edad": 30
        }
        ```
    
    Example Response:
        ```json
        {
            "id": 123,
            "email": "usuario@example.com",
            "nombre": "Juan Pérez",
            "edad": 30,
            "created_at": "2024-03-20T10:30:00Z"
        }
        ```
    """
```

## Reglas Generales

1. **Claridad sobre brevedad**: Mejor explicar de más que de menos
2. **Mantener actualizado**: Si cambias código, actualiza comentarios
3. **Evitar obviedades**: No comentar `i += 1  # Incrementa i en 1`
4. **Usar español neutro**: Sin modismos regionales
5. **Consistencia**: Mismo estilo en todo el proyecto
6. **Gramática correcta**: Revisar ortografía y puntuación

## Plantilla Rápida para Funciones Simples

```python
def funcion_simple(x: int) -> int:
    """
    [Verbo en presente] + [qué hace] + [con qué/para qué].
    
    Args:
        x: [Descripción breve del parámetro].
    
    Returns:
        [Qué retorna].
    """
    # [Explicación de paso complejo si existe]
    return x * 2
```

## Adaptaciones por Lenguaje

### JavaScript/TypeScript (JSDoc):
```javascript
/**
 * Descripción breve de la función.
 * 
 * Descripción detallada del propósito y comportamiento.
 * 
 * @param {string} param1 - Descripción del parámetro
 * @param {number} param2 - Otro parámetro
 * @returns {Promise<Object>} Descripción del retorno
 * @throws {Error} Cuándo se lanza error
 * 
 * @example
 * const resultado = await miFuncion('test', 123);
 * console.log(resultado);
 */
```

### Java (JavaDoc):
```java
/**
 * Descripción breve del método.
 * 
 * Descripción detallada del comportamiento.
 * 
 * @param param1 Descripción del parámetro
 * @param param2 Otro parámetro
 * @return Descripción del retorno
 * @throws TipoExcepcion Cuándo se lanza
 * 
 * @see ClaseRelacionada
 * @since 1.0.0
 */