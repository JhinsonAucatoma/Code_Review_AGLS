## Estado real en GitHub (verificado)

Ya se crearon 13 issues en `https://github.com/LEO23as/sga-sistema-distribuido/issues`. Mapeo confirmado contra la API pública:

| Este plan | GitHub | Estado |
|---|---|---|
| Issue 1 | [#2](https://github.com/LEO23as/sga-sistema-distribuido/issues/2) | ✅ correcto |
| Issue 2 | [#3](https://github.com/LEO23as/sga-sistema-distribuido/issues/3) | ✅ correcto |
| Issue 3 | [#4](https://github.com/LEO23as/sga-sistema-distribuido/issues/4) | ✅ correcto |
| Issue 4 | [#5](https://github.com/LEO23as/sga-sistema-distribuido/issues/5) | ✅ correcto |
| Issue 5 | [#6](https://github.com/LEO23as/sga-sistema-distribuido/issues/6) | ✅ correcto |
| Issue 6 | [#7](https://github.com/LEO23as/sga-sistema-distribuido/issues/7) | ✅ correcto |
| Issue 7 | [#8](https://github.com/LEO23as/sga-sistema-distribuido/issues/8) | ✅ correcto |
| Issue 8 | [#9](https://github.com/LEO23as/sga-sistema-distribuido/issues/9) | ✅ correcto |
| Issue 9 | [#11](https://github.com/LEO23as/sga-sistema-distribuido/issues/11) | ✅ correcto (no es #10) |
| Issue 10 | [#12](https://github.com/LEO23as/sga-sistema-distribuido/issues/12) | ✅ correcto |
| Issue 11 | [#13](https://github.com/LEO23as/sga-sistema-distribuido/issues/13) | ✅ correcto |
| Issue 12 | [#14](https://github.com/LEO23as/sga-sistema-distribuido/issues/14) | ✅ correcto |
| Issue 13 (sugerencia) | — | ❌ **no se creó** |
| — | [#10](https://github.com/LEO23as/sga-sistema-distribuido/issues/10) | ⚠️ duplicado exacto del #9 (mismo título) |

**Acción pendiente:** editar el issue **#10** en GitHub (ícono de lápiz en título y cuerpo) y reemplazar su contenido por el del **Issue 13** de este documento ("Limpieza de repositorio..."), en vez de cerrarlo como duplicado — así no se pierde el número ni queda un hallazgo (H14) sin issue.

**También pendiente:** ningún issue tiene *label* ni *assignee* todavía. Crear las etiquetas indicadas en cada bloque de abajo y asignar cada issue a un integrante de BCEL.

---

# Plan de issues — Revisión cruzada AGLS → BCEL (SGA Escuela Provincias Unidas)

Este archivo contiene los 13 issues que el equipo **AGLS** debe abrir en el repositorio de **BCEL**
(`https://github.com/LEO23as/sga-sistema-distribuido`), siguiendo la plantilla del Anexo A de FORM-TL-05.

## Cómo usar cada bloque en GitHub

Cada issue de abajo trae tres partes separadas, pensadas para los tres lugares distintos de la
pantalla de "New issue" de GitHub:

1. **Título** → pegar tal cual en el campo *Title*. No lleva `##` ni `Issue #N —`: eso era solo
   la organización de este documento, GitHub numera los issues automáticamente y ese número casi
   seguro no va a coincidir con el de esta lista (no pasa nada, usa el número real que te asigne
   GitHub cuando actualices la Sección 4 del informe).
2. **Etiquetas** → no se pegan en ningún campo de texto. Se seleccionan en el panel derecho,
   sección **Labels** (ícono de engranaje). Créalas la primera vez si no existen:
   `solid`, `patrones`, `capas`, `excepciones`, `logging`, `seguridad`,
   `severidad:critica`, `severidad:mayor`, `severidad:menor`, `sugerencia`.
3. **Cuerpo** → pegar tal cual en el campo de descripción (debajo del título). La primera línea
   ya trae el Bloque/Item y un espacio para el nombre del revisor de AGLS; reemplaza
   `[Nombre revisor]` antes de pegar, o edítalo después de crear el issue.

Además, en el panel derecho de GitHub: asigna **Assignee** a un integrante de **BCEL** (nunca de
AGLS), y anota la URL final del issue en la Sección 4 (Tabla de issues abiertos) de
`informe_revision.tex`.

---

## Issue 1

**Título:**
```
Extraer el parseo de archivos de ImportacionExcelService en estrategias por formato
```

**Etiquetas:** `solid`, `severidad:mayor` &nbsp;·&nbsp; **Bloque/Item:** A1 (SRP)

**Cuerpo:**
```markdown
**Bloque/Item:** A1 (SRP) · **Revisor:** [Nombre revisor]

**Contexto:** `ImportacionExcelService` (`sga-principal/src/main/java/ec/edu/uteq/sga/service/ImportacionExcelService.java`, 388 líneas) concentra el parseo de tres formatos de archivo completamente distintos (CSV vía Apache Commons CSV, Excel vía Apache POI, PDF vía Apache PDFBox con extracción heurística por expresiones regulares) y además ejecuta la persistencia final de `Estudiante` y `Matricula` en el mismo método `confirmarImportacion` (líneas 319-387).

**Evidencia:**
\`\`\`java
// ImportacionExcelService.java:34-39 (comentario del propio equipo BCEL)
/**
 * DEUDA TECNICA CONOCIDA: mismo bypass que ImportacionCasService (escribe
 * estudiantes/matriculas por JPA directo, sin pasar por sga-secretaria).
 * Se acepta a proposito para este flujo de carga masiva inicial...
 */
\`\`\`

**Diagnóstico:** La clase tiene más de una razón para cambiar: un cambio en el formato de un archivo PDF de listados CAS, un cambio en las columnas esperadas del Excel, o un cambio en cómo se persiste la matrícula, obligan a modificar el mismo archivo. Esto viola el Principio de Responsabilidad Única.

**Propuesta de corrección:**
1. Definir una interfaz `ImportadorArchivo { List<Map<String,String>> parsear(MultipartFile archivo); boolean soporta(String nombreArchivo); }`.
2. Mover `leerCsv`, `leerExcel` y `leerPdf` a implementaciones separadas (`CsvImportador`, `ExcelImportador`, `PdfCasImportador`).
3. Extraer la persistencia de `confirmarImportacion` a un `ImportacionEstudiantesWriter` independiente del parseo.
4. `ImportacionExcelService` queda como orquestador delgado que resuelve el importador correcto y delega.

**Criterio de aceptación:** cada formato de archivo puede probarse unitariamente sin instanciar los otros dos parsers; `ImportacionExcelService` no supera ~80 líneas.
```

---

## Issue 2

**Título:**
```
Sustituir el if/else de parsearArchivo por un registro de ImportadorArchivo
```

**Etiquetas:** `solid`, `severidad:menor` &nbsp;·&nbsp; **Bloque/Item:** A2 (OCP)

**Cuerpo:**
```markdown
**Bloque/Item:** A2 (OCP) · **Revisor:** [Nombre revisor]

**Contexto:** `parsearArchivo` (`ImportacionExcelService.java:61-70`) decide el parser con una cadena `if/else if` sobre la extensión del nombre de archivo.

**Evidencia:**
\`\`\`java
if (nombreArchivo.endsWith(".csv")) {
    filas = leerCsv(archivo);
} else if (nombreArchivo.endsWith(".xlsx") || nombreArchivo.endsWith(".xls")) {
    filas = leerExcel(archivo);
} else if (nombreArchivo.endsWith(".pdf")) {
    filas = leerPdf(archivo);
} else {
    throw new ResponseStatusException(HttpStatus.BAD_REQUEST, ...);
}
\`\`\`

**Diagnóstico:** Agregar soporte para un formato nuevo (p. ej. `.ods`) exige editar este método existente en vez de solo añadir código nuevo, violando el Principio Abierto/Cerrado.

**Propuesta de corrección:** Depende de la refactorización del Issue 1. Con la interfaz `ImportadorArchivo`, inyectar `List<ImportadorArchivo>` y resolver con `.stream().filter(i -> i.soporta(nombreArchivo)).findFirst()`. Agregar un formato nuevo se reduce a registrar un nuevo bean, sin tocar `ImportacionExcelService`.

**Criterio de aceptación:** añadir un importador de prueba (`.txt`) no requiere modificar ninguna clase existente, solo agregar una nueva.
```

---

## Issue 3

**Título:**
```
Enrutar AuthController exclusivamente a través de UsuarioService
```

**Etiquetas:** `solid`, `capas`, `severidad:mayor` &nbsp;·&nbsp; **Bloque/Item:** A5 (DIP) / C2

**Cuerpo:**
```markdown
**Bloque/Item:** A5 (DIP) / C2 (dirección de dependencias) · **Revisor:** [Nombre revisor]

**Contexto:** `AuthController` (`sga-principal/src/main/java/ec/edu/uteq/sga/controller/AuthController.java`) inyecta `UsuarioRepository` directamente y lo consulta sin manejar la ausencia de resultado.

**Evidencia:**
\`\`\`java
// AuthController.java:9,23,39-41
import ec.edu.uteq.sga.repository.UsuarioRepository;
...
private final UsuarioRepository usuarioRepository;
...
Usuario usuario = usuarioRepository.findByUsername(request.getUsername())
        .orElseThrow();
\`\`\`

**Diagnóstico:** El controlador (capa de presentación) depende directamente del repositorio (capa de persistencia), saltando `UsuarioService`. Además, `.orElseThrow()` sin argumento lanza `NoSuchElementException`, que no está mapeada a ningún `@ExceptionHandler`, por lo que el cliente recibe un error 500 genérico ante un usuario inexistente.

**Propuesta de corrección:**
1. Añadir un método `UsuarioService.buscarPorUsername(String username)` que encapsule la búsqueda y lance una excepción de negocio clara (`ResponseStatusException(UNAUTHORIZED, "Credenciales inválidas")`).
2. `AuthController` deja de inyectar `UsuarioRepository` y solo depende de `UsuarioService` y `JwtUtil`.

**Criterio de aceptación:** `AuthController` no importa ninguna clase del paquete `repository`; un intento de login con usuario inexistente devuelve un JSON de error consistente con el resto del API, no una traza genérica.
```

---

## Issue 4

**Título:**
```
Dividir PrincipalGrpcService en adaptadores por agregado que solo llamen a servicios
```

**Etiquetas:** `solid`, `patrones`, `capas`, `severidad:mayor` &nbsp;·&nbsp; **Bloque/Item:** A1/A5, C2, B3

**Cuerpo:**
```markdown
**Bloque/Item:** A1/A5 (SRP/DIP) · C2 (capas) · B3 (patrón mal aplicado) · **Revisor:** [Nombre revisor]

**Contexto:** `PrincipalGrpcService` (`sga-principal/src/main/java/ec/edu/uteq/sga/grpc/PrincipalGrpcService.java`, 382 líneas) es el único punto de entrada gRPC que consultan `microservicio-secretaria` y `microservicio-soporte`. Inyecta 4 repositorios (`AnoLectivoRepository`, `AsignaturaRepository`, `RepresentanteRepository`, `EstudianteRepository`) además de 2 servicios (`EstudianteService`, `GradoService`).

**Evidencia:**
\`\`\`java
// PrincipalGrpcService.java:37-44
public class PrincipalGrpcService extends PrincipalServiceGrpc.PrincipalServiceImplBase {
    private final AnoLectivoRepository anoLectivoRepository;
    private final AsignaturaRepository asignaturaRepository;
    private final RepresentanteRepository representanteRepository;
    private final EstudianteRepository estudianteRepository;
    // ... además de EstudianteService y GradoService
\`\`\`

**Diagnóstico:** La capa de comunicación remota mezcla marshalling de protobuf, acceso a datos directo y reglas de negocio de cinco agregados distintos (año lectivo, asignatura, representante, estudiante, grado). Cualquier cambio en cómo se accede a un agregado tiene radio de impacto en dos microservicios externos que dependen del mismo `.proto`.

**Propuesta de corrección:**
1. Crear un servicio de aplicación por agregado que sea la única puerta de entrada tanto para REST como para gRPC (p. ej. `EstudianteService` ya existe: usarlo también para las operaciones de estudiante en gRPC; crear los que falten para grado/asignatura/representante/año lectivo).
2. `PrincipalGrpcService` deja de inyectar repositorios; solo inyecta los servicios de aplicación y traduce DTO ↔ protobuf.
3. Documentar en el `.proto` o en un ADR qué servicio es dueño de cada operación.

**Criterio de aceptación:** `PrincipalGrpcService` no importa ninguna clase del paquete `repository`; cada RPC delega en un único método de un servicio de aplicación.
```

---

## Issue 5

**Título:**
```
Aislar el esquema de microservicio-docente (rol de BD sin acceso a sga_principal)
```

**Etiquetas:** `capas`, `severidad:critica` &nbsp;·&nbsp; **Bloque/Item:** B5, C5 — CRÍTICO

**Cuerpo:**
```markdown
**Bloque/Item:** B5 (antipatrón de BD compartida) / C5 (configuración) · **Revisor:** [Nombre revisor]

**Contexto:** La conexión de `microservicio-docente` a PostgreSQL se configura con un `search_path` que incluye el esquema de otro servicio.

**Evidencia:**
\`\`\`python
# micro_docente/settings.py:53-65
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "HOST": os.environ.get("DB_HOST", "3.23.195.43"),
        "PORT": os.environ.get("DB_PORT", "5433"),
        "OPTIONS": {
            "options": "-c search_path=sga_docente,sga_principal,public",
        },
    }
}
\`\`\`

**Diagnóstico:** Esta es la configuración de conexión por defecto, no una consulta aislada: cualquier query ORM de `docente` puede alcanzar tablas de `sga_principal` sin pasar por el gRPC diseñado para ese propósito. Sumado a que los 4 backends convergen en la misma instancia física de PostgreSQL (`3.23.195.43:5433`), el sistema no cumple *database per service* pese a describirse como "persistencia de datos distribuida" en el `README.md`. Es el hallazgo de mayor riesgo de integridad de datos del sistema.

**Propuesta de corrección:**
1. Crear un rol de PostgreSQL específico para `microservicio-docente` con `search_path=sga_docente,public` únicamente y sin permisos `GRANT` sobre el esquema `sga_principal`.
2. Auditar el código de `docentes/` en busca de queries que dependan del acceso implícito a `sga_principal` y migrarlas al cliente gRPC existente (`grpc_services/client.py`).
3. Repetir la auditoría de `search_path`/roles para `secretaria` y `soporte`.

**Criterio de aceptación:** una consulta de prueba desde el rol de BD de `docente` contra una tabla de `sga_principal` debe fallar por permisos, no solo por convención de código.
```

---

## Issue 6

**Título:**
```
Externalizar y rotar todos los secretos versionados (BD, JWT, SMTP, token gRPC)
```

**Etiquetas:** `seguridad`, `severidad:critica` &nbsp;·&nbsp; **Bloque/Item:** C5 — CRÍTICO

**Cuerpo:**
```markdown
**Bloque/Item:** C5 (configuración externalizada) · **Revisor:** [Nombre revisor]

**Contexto:** Contraseña de base de datos, secreto JWT, contraseña de aplicación de Gmail y token interno de gRPC están hardcodeados como valor por defecto funcional (no de ejemplo) y versionados en texto claro. El `README.md` además publica credenciales reales de acceso.

**Evidencia:**
\`\`\`properties
# sga-principal/src/main/resources/application.properties:10,29,33-34
spring.datasource.password=${DB_PASSWORD:SgaProvU2026Db}
jwt.secret=sga-provincias-unidas-secret-key-2026-ecuador-uteq-sistemas
spring.mail.username=kbedonv@uteq.edu.ec
spring.mail.password=wftctyvpwbxwpfhe
\`\`\`
\`\`\`java
// InternalAuthInterceptor.java:14-15
// TODO: Usar variable de entorno para producción, hardcodeado por ahora como "dev-token-123"
private static final String EXPECTED_TOKEN = "dev-token-123";
\`\`\`
El mismo `DB_PASSWORD` por defecto se repite en `microservicio-docente/micro_docente/settings.py:58`, y el `internal_token` hardcodeado se repite en `ActividadGrpcClient.java:26`. El `README.md` publica usuario/contraseña reales de administrador y docente.

**Diagnóstico:** El patrón `${VAR:default}` de Spring da una falsa sensación de externalización: si no se define la variable de entorno, el sistema arranca igualmente con el secreto real versionado. Cualquiera con acceso de lectura al repositorio (incluido este ejercicio de revisión cruzada) tiene la contraseña de la base de datos de producción, el secreto de firma JWT y la contraseña de la cuenta de Gmail institucional.

**Propuesta de corrección:**
1. Quitar todos los valores por defecto reales de `application.properties`/`settings.py`; usar `${VAR}` sin default, o un default claramente inválido que falle rápido si no se configura.
2. Rotar inmediatamente: contraseña de PostgreSQL, `jwt.secret`, contraseña de aplicación de Gmail, `internal_token` de gRPC.
3. Quitar la tabla de credenciales reales del `README.md`; reemplazarla por instrucciones para obtener credenciales de prueba por un canal privado (o crear usuarios semilla con contraseña aleatoria de un solo uso).
4. Añadir `application.properties`/`settings.py` con secretos a `.gitignore` y usar un `.properties.example` sin valores reales.

**Criterio de aceptación:** `git log -p -- '**/application.properties' '**/settings.py'` no debe mostrar ningún secreto real después de la rotación; `README.md` no contiene contraseñas reales.
```

---

## Issue 7

**Título:**
```
Añadir deadline, política de reintento acotada y degradación a los clientes gRPC
```

**Etiquetas:** `excepciones`, `severidad:mayor` &nbsp;·&nbsp; **Bloque/Item:** D2/D3/D4

**Cuerpo:**
```markdown
**Bloque/Item:** D2 / D3 / D4 · **Revisor:** [Nombre revisor]

**Contexto:** Ningún cliente gRPC del sistema (`ActividadGrpcClient`, `AsistenciaGrpcClient`, `DocenteGrpcClient` en `sga-principal`; `PrincipalGrpcClient` en `secretaria`; `TecnicoGrpcClient` en `soporte`) fija `withDeadlineAfter`. Las llamadas son bloqueantes sin límite de espera propio, sin reintento y sin valor de reserva ante fallo.

**Evidencia:**
\`\`\`java
// ActividadGrpcClient.java:30-37
public ActividadResponse crearActividad(CrearActividadRequest request) {
    try {
        return getStubWithMetadata().crearActividad(request);
    } catch (StatusRuntimeException e) {
        System.err.println("Error gRPC crearActividad: " + e.getMessage());
        throw e;
    }
}
\`\`\`

**Diagnóstico:** Si `microservicio-docente` deja de responder (proceso vivo pero colgado), el hilo HTTP de Tomcat que atiende la petición de `sga-principal` queda bloqueado indefinidamente. Bajo varias peticiones concurrentes al mismo endpoint, el pool de hilos se agota y `sga-principal` deja de responder también a peticiones que no dependían de `docente` — una falla en cascada clásica de sistemas distribuidos.

**Propuesta de corrección:**
1. Fijar un `Deadline` explícito en cada llamada: `stub.withDeadlineAfter(3, TimeUnit.SECONDS)`.
2. Introducir Resilience4j (u otra librería de circuit breaker) envolviendo los 5 clientes gRPC, con un umbral de fallos que abra el circuito.
3. Definir, para cada operación, un valor de degradación razonable cuando el circuito está abierto (p. ej. lista vacía con bandera `degradado: true` para operaciones de lectura; error explícito y claro para escrituras, sin reintento automático de escrituras no idempotentes).

**Criterio de aceptación:** una prueba que detiene `microservicio-docente` y hace 20 peticiones concurrentes a `sga-principal` no debe agotar el pool de hilos ni afectar a endpoints no relacionados con `docente`.
```

---

## Issue 8

**Título:**
```
Unificar el formato de error con un @ControllerAdvice en sga-principal
```

**Etiquetas:** `excepciones`, `severidad:mayor` &nbsp;·&nbsp; **Bloque/Item:** D5

**Cuerpo:**
```markdown
**Bloque/Item:** D5 · **Revisor:** [Nombre revisor]

**Contexto:** `microservicio-secretaria` y `microservicio-soporte` tienen un `GlobalExceptionHandler` (`@RestControllerAdvice`) propio que normaliza el formato de error JSON. `sga-principal` no tiene ningún manejador global y depende de `ResponseStatusException` ad-hoc por método de servicio.

**Evidencia:** `grep -rn "@RestControllerAdvice\|@ControllerAdvice" sga-principal/src/main/java` no devuelve resultados, frente a `microservicio-soporte/backend/src/main/java/ec/uteq/sga/soporte/common/GlobalExceptionHandler.java` (127 líneas, maneja `MethodArgumentNotValidException`, `DataAccessException`, `Exception` genérica con cuerpo `{"error": ..., "detalle": ...}`).

**Diagnóstico:** Un error no cubierto explícitamente por un `ResponseStatusException` en `sga-principal` (por ejemplo, el `NoSuchElementException` de `AuthController.java:41`, ver Issue 3) cae en el manejador de error por defecto de Spring Boot, cuyo formato no coincide con el de `secretaria`/`soporte`. Un frontend que consuma los 4 backends debe manejar 2 formatos de error distintos.

**Propuesta de corrección:** Portar `GlobalExceptionHandler` de `microservicio-soporte` a `sga-principal`, adaptando los tipos de excepción específicos de JPA/Hibernate que use `sga-principal` (además de las ya cubiertas: `MethodArgumentNotValidException`, `DataAccessException`, `Exception`). Considerar extraer un starter/librería compartida si el equipo BCEL mantiene los 4 servicios a futuro.

**Criterio de aceptación:** una petición inválida a cualquiera de los 4 backends devuelve un cuerpo JSON con la misma forma (`error`, `detalle`/`detalles`).
```

---

## Issue 9

**Título:**
```
Reemplazar System.out/err.println por el logger inyectado en toda la base
```

**Etiquetas:** `logging`, `severidad:menor` &nbsp;·&nbsp; **Bloque/Item:** E1

**Cuerpo:**
```markdown
**Bloque/Item:** E1 · **Revisor:** [Nombre revisor]

**Contexto:** Solo 4 clases de `sga-principal` usan `@Slf4j` (`EmailService`, `ImportacionCasService`, `ImportacionExcelService`, `UsuarioService`). El resto de puntos de depuración usa `System.out`/`err.println`, incluido el propio punto de entrada de la aplicación.

**Evidencia:**
\`\`\`java
// SgaPrincipalApplication.java:12-16
public static void main(String[] args) {
    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    System.out.println("HASH: " + encoder.encode("Admin1234"));
    SpringApplication.run(SgaPrincipalApplication.class, args);
}
\`\`\`
También en `DocenteContextController.java:31-86`, `DataSeedRunner.java:47-182` y `ActividadGrpcClient.java:34,43,53,68,78`.

**Diagnóstico:** Los mensajes por `System.out`/`err` no pasan por ningún appender configurado, no tienen nivel de severidad, no se pueden filtrar ni redirigir a un agregador de logs, y en el caso del `main()` imprimen además un dato sensible (ver Issue 11).

**Propuesta de corrección:** Reemplazar todos los `System.out`/`err.println` por `@Slf4j log.debug/info/warn/error` con el nivel apropiado; eliminar por completo la línea de `main()` que imprime el hash (no cumple ningún propósito en producción).

**Criterio de aceptación:** `grep -rn "System\.\(out\|err\)\.print" sga-principal/src/main` no devuelve resultados fuera de pruebas.
```

---

## Issue 10

**Título:**
```
Propagar un identificador de correlación entre HTTP y gRPC
```

**Etiquetas:** `logging`, `severidad:mayor` &nbsp;·&nbsp; **Bloque/Item:** E2/E3

**Cuerpo:**
```markdown
**Bloque/Item:** E2 / E3 · **Revisor:** [Nombre revisor]

**Contexto:** No existe ningún identificador de correlación en el sistema. Los metadatos gRPC que sí se propagan (`docente_id`, `internal_token`) autentican al llamante pero no identifican la petición.

**Evidencia:** `grep -rln "MDC\|correlationId\|traceId\|X-Request-Id"` sobre `sga-principal/`, `microservicio-secretaria/`, `microservicio-soporte/`, `microservicio-docente/` no devuelve resultados.

**Diagnóstico:** Sin un identificador compartido, no es posible reconstruir la traza de una operación que cruza `sga-principal` → gRPC → `microservicio-docente` a partir de los registros actuales; solo queda correlacionar por proximidad de marca de tiempo, lo cual no es fiable bajo carga concurrente.

**Propuesta de corrección:**
1. Añadir un filtro Servlet en cada backend Java que genere (o propague, si ya viene en el header `X-Correlation-Id`) un UUID por petición y lo coloque en `MDC`.
2. Incluir ese UUID como metadato gRPC adicional (junto a `internal_token`) en los 5 clientes gRPC, y leerlo del lado servidor para volver a ponerlo en `MDC` antes de procesar la llamada.
3. Incluir `%X{correlationId}` en el patrón de logging de todos los servicios (incluido Django, vía `logging` de Python con un `Filter` equivalente).

**Criterio de aceptación:** una petición HTTP a `sga-principal` que dispara una llamada a `microservicio-docente` produce, en ambos servicios, al menos un registro con el mismo identificador de correlación.
```

---

## Issue 11

**Título:**
```
Eliminar la impresión del hash de contraseña en main() y las credenciales del README.md
```

**Etiquetas:** `logging`, `seguridad`, `severidad:critica` &nbsp;·&nbsp; **Bloque/Item:** E4 — CRÍTICO

**Cuerpo:**
```markdown
**Bloque/Item:** E4 · **Revisor:** [Nombre revisor]

**Contexto:** `main()` imprime en cada arranque el hash bcrypt de una contraseña hardcodeada; el `README.md` publica usuario y contraseña reales de administrador y docente en texto claro.

**Evidencia:**
\`\`\`java
// SgaPrincipalApplication.java:12-16
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
System.out.println("HASH: " + encoder.encode("Admin1234"));
\`\`\`
\`\`\`markdown
<!-- README.md, sección "Credenciales de Acceso para Evaluación" -->
| Administrador | pcastrol2 | 402/42745aA | ... |
| Docente Titular | jsjimenezt | 402/42745aA | ... |
\`\`\`

**Diagnóstico:** Aunque el propósito original parece ser generar un hash de prueba durante desarrollo, el código quedó en el punto de entrada de producción y se ejecuta en cada arranque, exponiendo en los logs de despliegue tanto la contraseña en texto claro como su hash. El `README.md`, al ser público en GitHub, expone credenciales reales de dos roles del sistema en producción.

**Propuesta de corrección:**
1. Eliminar por completo la línea `System.out.println("HASH: ...")` de `main()`.
2. Quitar la tabla de credenciales reales del `README.md`. Si se necesita un ambiente de evaluación, documentar cómo crear un usuario semilla con contraseña generada aleatoriamente y de un solo uso, o compartir credenciales de prueba por un canal privado (no en el repositorio).
3. Confirmar que la contraseña `Admin1234` no está en uso real en ningún ambiente; si lo está, forzar cambio de contraseña.

**Criterio de aceptación:** el log de arranque de `sga-principal` no contiene ningún hash ni contraseña; `README.md` no contiene credenciales reales.
```

---

## Issue 12

**Título:**
```
Ampliar HAProxy (o introducir un API Gateway) para los tres microservicios restantes
```

**Etiquetas:** `patrones`, `severidad:menor` &nbsp;·&nbsp; **Bloque/Item:** B4

**Cuerpo:**
```markdown
**Bloque/Item:** B4 · **Revisor:** [Nombre revisor]

**Contexto:** La única puerta de enlace del sistema (HAProxy) balancea exclusivamente `sga-principal` (REST 8080 y gRPC 9092).

**Evidencia:**
\`\`\`
# infra/haproxy/haproxy.cfg:24-47
frontend fe_principal
    bind *:8080
    default_backend be_principal
backend be_principal
    balance roundrobin
    server-template principal 2 sga-principal:8080 check inter 3s fall 2 rise 2
# (no existen frontend/backend equivalentes para docente, secretaria o soporte)
\`\`\`

**Diagnóstico:** `microservicio-docente`, `microservicio-secretaria` y `microservicio-soporte` no tienen entrada en HAProxy, por lo que el frontend debe conocer y llamar directamente cada puerto de servicio (ver tabla de puertos del `README.md`). Esto impide un punto único de entrada, dificulta aplicar políticas transversales (rate limiting, TLS termination, autenticación) y hace más frágil cualquier cambio de topología de red.

**Propuesta de corrección:** Añadir `frontend`/`backend` en `haproxy.cfg` para los tres servicios restantes, replicando el patrón de `health check` ya usado para `sga-principal` (`httpchk GET /actuator/health/liveness` o el endpoint de salud equivalente en Django/Express). Evaluar a mediano plazo introducir un API Gateway real (p. ej. Spring Cloud Gateway) que además centralice autenticación JWT.

**Criterio de aceptación:** los tres microservicios responden a través de HAProxy con el mismo patrón de `health check` que `sga-principal`.
```

---

## Issue 13

**Título:**
```
Limpieza de repositorio: clase vacía, binarios de target/ versionados, .gitignore corrupto
```

**Etiquetas:** `sugerencia` &nbsp;·&nbsp; **Bloque/Item:** Sugerencia (agrupa varios hallazgos menores)

**Cuerpo:**
```markdown
**Bloque/Item:** Sugerencia · **Revisor:** [Nombre revisor]

**Contexto y evidencia:**
1. Clase pública vacía con nombre en minúscula: `sga-principal/src/main/java/ec/edu/uteq/sga/repository/repository.java` contiene solo `public class repository {}`, violando la convención de nombres de Java y sin ningún uso.
2. Binarios de compilación versionados pese a que `.gitignore` excluye `**/target/`: `microservicio-soporte/backend/target/protoc-plugins/protoc-4.28.2-windows-x86_64.exe` y `protoc-gen-grpc-java-1.68.1-windows-x86_64.exe` están presentes en el árbol de trabajo entregado.
3. `.gitignore` con codificación corrupta en sus últimas líneas (aparece como `. c l a u d e /` con espacios/UTF-16, en vez de `.claude/`), por lo que probablemente no excluye lo que se pretendía.
4. Documentación desactualizada: el `README.md` describe `microservicio-secretaria` y `microservicio-soporte` como "Node.js/Express"; el código real es Java 17/Spring Boot (Maven, `pom.xml`).

**Propuesta de corrección:**
1. Eliminar `repository.java`.
2. Confirmar si los binarios de `target/` fueron agregados con `git add -f` antes de existir el `.gitignore`, y purgarlos del árbol de trabajo (evaluar si conviene reescribir historia con `git filter-repo` si son voluminosos).
3. Reescribir `.gitignore` en UTF-8 sin BOM y verificar con `git check-ignore -v` que excluye lo esperado.
4. Corregir la tabla de arquitectura del `README.md` para reflejar Spring Boot en `secretaria`/`soporte`.

**Criterio de aceptación:** `git status` tras un `git clone` limpio no muestra artefactos de compilación; `README.md` coincide con el stack real de cada servicio.
```
