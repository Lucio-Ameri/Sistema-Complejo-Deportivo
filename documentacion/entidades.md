# 1. CLASES ABSTRACTAS
#

## EntidadBase
Clase base para todas las entidades persistentes del sistema. Provee identidad única generada por la base de datos
y control de concurrencia optimista.

**Atributos:**

    - private Long id;
    - private Long version;

**Relaciones:**
- Ninguna. Clase raíz de la jerarquía de persistencia.

**Datos Importantes:**
- Anotación JPA: `@MappedSuperclass`.
- El atributo `id` se genera automáticamente por la estrategia de la base de datos (`@GeneratedValue(strategy = GenerationType.IDENTITY)`).
- Nunca exponer setter de `id`. Solo getter.
- Todas las entidades persistentes heredan de esta clase a través de `AuditoriaEntidad`.
- `version` está anotado con `@Version` (optimistic locking de JPA). Toda entidad mutable del sistema queda protegida contra
  *lost updates* en escrituras concurrentes sin necesidad de declarar el campo en cada subclase.
- Nunca exponer setter de `version`. Solo getter. Lo gestiona el proveedor de persistencia.
---

## AuditoriaEntidad (Extiende de EntidadBase)
Clase base para auditoría de negocio.

**Atributos:**

    - private LocalDateTime creadoEn;
    - private LocalDateTime actualizadoEn;
    - private Long creadoPorId;
    - private Long actualizadoPorId;

**Relaciones:**
- Ninguna directa. Las referencias a `Usuario` son por ID para evitar dependencias circulares entre Aggregates.

**Datos Importantes:**
- `@MappedSuperclass` para propagación JPA a todas las subclases.
- `creadoEn` / `actualizadoEn` se gestionan con `@PrePersist` / `@PreUpdate`.
- Referencias a `Usuario` son por ID para evitar dependencias circulares entre aggregates.
- `creadoPorId` puede ser `null` para procesos automáticos del sistema (generación de cuotas, backups automáticos).
---

## Persona (Extiende de AuditoriaEntidad)
Clase abstracta que representa datos comunes de cualquier persona física en el sistema.

**Atributos:**

    - private NombrePersona nombre;
    - private DocumentoIdentidad documento;
    - private Contacto contacto;
    - private String observaciones;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- `@MappedSuperclass`.
- `Cliente` y `Responsable` extienden de esta clase.
- `documento` se modela como obligatorio en `Persona`. `Cliente` puede relajar esta restricción mediante un factory
  method dedicado (`Cliente.sinDocumento(...)`) cuando el negocio así lo permita según `ConfiguracionSistema.preferencias.permitirRegistroSinDNI`.
- La validación diferenciada por subclase se gestiona en sus respectivos factory methods / constructores, no modificando
  el contrato base de `Persona`.
---

## OperacionBase (Extiende de AuditoriaEntidad)
Clase abstracta que representa una operación general del sistema con fecha y estado.

**Atributos**

    - private LocalDateTime fechaDeLaOperacion;
    - private EstadoOperacion estado;
    - private String observaciones;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- No representa dinero. No todas las operaciones son económicas.
- `EstadoOperacion`: `ACTIVA`, `ANULADA`.
- Las subclases NO agregan un segundo campo de estado que solape con `estado` de esta clase. Si necesitan estados
  más específicos, estos se declaran como campos adicionales con semántica distinta (Ej.: `EstadoReserva` es el estado
  del proceso de reserva, no del ciclo de vida de la operación). La anulación siempre usa `estado = ANULADA`.
- `Pago` extiende de esta clase.
---

## OperacionCobrableBase (Extiende de OperacionBase e implementa Cobrable y ReceptorDePago)
Clase abstracta para operaciones que generan un importe a cobrar.

**Atributos**

    - private Dinero importeTotal;
    - private Dinero importePagado;
    - private EstadoCobro estadoCobro;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Representa algo que **debe ser pagado**, no un pago en sí mismo.
- `estadoCobro`: `PENDIENTE`, `PARCIAL`, `SALDADO`, `ANULADO`.
- `importePagado` se actualiza al registrar `AplicacionPago` vía el método `registrarAplicacionPago()` de la interfaz
  `ReceptorDePago`. El recálculo de `estadoCobro` ocurre dentro de este método.
---

## OperacionConEspacioFisico (Extiende de OperacionCobrableBase)
Clase abstracta intermedia para operaciones que requieren ocupar un espacio físico. Elimina duplicación entre `Reserva`
y `Evento`.

**Atributos:**

    - private Long ocupacionId;
    - private ParticipanteOperacion participante;

**Relaciones:**
- Ninguna directa (referencia a cliente contenida dentro de `ParticipanteOperacion`).

**Datos Importantes:**
- Extrae los atributos comunes de `Reserva` y `Evento`.
- `OcupacionEspacio` es la entidad de bloqueo físico (aggregate aparte). La operación comercial la referencia por
  `ocupacionId`, no la contiene, manteniendo bajo acoplamiento entre aggregates.
- `participante` reemplaza el patrón de campo nullable `cliente` / `eventual`. Incluye la referencia al cliente cuando aplica.
- No se declara `clienteId` como atributo propio: esa información ya vive en `participante.clienteId`.
---

## RangoHorario
Clase base abstracta que representa un período de tiempo con inicio y fin.

**Atributos:**

    - private LocalTime horaDeInicio;
    - private LocalTime horaDeFin;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Sus subclases `HorarioConcreto` y `HorarioRecurrente` son `@Embeddable`.
- Validar en constructor que `horaDeFin` sea estrictamente posterior a `horaDeInicio`.
---
#
#
#
#
#
# 2. VALUE OBJECTS
#
## Dinero (@Embeddable)
Value Object para importes monetarios.

**Atributos:**

    - private BigDecimal monto;
    - private Moneda tipo;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- `monto` con escala 2 y `RoundingMode.HALF_UP`. Nunca usar `double` ni `float`.
- `MONEDA`: `ARS`, `EURO`, `DOLAR`.
- Validar en constructor: `monto` no puede ser `null`. La negatividad se valida contextualmente (puede haber ajustes
  negativos en caja pero no en cuotas).
- Constante útil: `Dinero.CERO` = `new Dinero(BigDecimal.ZERO, Moneda.ARS)`.
- Métodos útiles: `sumar(Dinero)`, `restar(Dinero)`, `esMayorQue(Dinero)`, `esIgualA(Dinero)`, `esCero()`.
- `sumar` y `restar` validan que ambas instancias tengan la misma `Moneda`. Lanzar `MonedaIncompatibleException` si difieren.
- Implementar `equals` y `hashCode` por valor (`monto` + `moneda`).
---

## PeriodoMensual (@Embeddable)
Value Object para representar el período mensual de una cuota.

**Atributos:**

    - private final YearMonth periodo;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Inmutable.
- La deuda del cliente se mide en meses, no en días exactos.
- Métodos: `anterior()`, `siguiente()`, `esPosteriorA(PeriodoMensual)`.
- Implementar `equals`, `hashCode` y `Comparable< PeriodoMensual >`.
- La restricción de unicidad `clienteId + inscripcionId + periodo` es responsabilidad de la entidad `Cuota`, no de este
  Value Object. Ver `Cuota` para la documentación completa de dicha restricción y su justificación de diseño.
---

## DocumentoIdentidad (@Embeddable)
Value Object para DNI u otro documento de identidad.

**Atributos:**

    - private TipoDocumento tipo;
    - private String numero;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Permite normalizar búsqueda y detección de duplicados.
- `numero` sin espacios ni puntos (normalizado en constructor: `numero.replaceAll("[^0-9A-Za-z]", "")`).
- `TipoDocumento`: `DNI`, `CUIT`, `PASAPORTE`, `LE`, `LC`.
- La opcionalidad del documento es responsabilidad de la entidad que lo usa (`Cliente` puede ser sin documento si la
  configuración del sistema lo permite; `Responsable` también lo admite opcional según el Flujo 1, paso 21).
- Implementar `equals` y `hashCode` por valor (`tipo` + `numero` normalizado).
---

## NombrePersona (@Embeddable)
Value Object para nombre/apellido normalizado.

**Atributos:**

    - private String nombre;
    - private String apellido;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Constructor rechaza strings vacíos o nulos para ambos campos.
- Normalización: trim y capitalización en constructor.
- Método `nombreCompleto()`: devuelve `apellido + ", " + nombre`.
- Útil para ordenamiento: `Comparator.comparing(p -> p.getNombre().getApellido())`.
- Implementar `equals` y `hashCode` por valor.
---

## Contacto (@Embeddable)
Value Object para agrupar datos comunes de contacto.

**Atributos:**

    - private String telefono;
    - private String email;
    - private String domicilio;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Validar formato de email con regex estándar cuando se informe (no nulo, contiene `@`).
- Todos los campos son opcionales (puede existir un contacto solo con teléfono).
- Implementar `equals` y `hashCode` por valor.
---

## DatosPersonaEventual (@Embeddable)
Datos mínimos de una persona que no se registra como cliente permanente, como aquel que alquila la cancha de fútbol una
vez, o alguien que alquila el salon infantil por unica vez, etc.

**Atributos:**

    - private NombrePersona nombre;
    - private Contacto contacto;
    - private String observaciones;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Para personas que realizan una operación puntual sin registrarse como clientes permanentes.
- Al menos `nombre` es obligatorio. `contacto` puede ser vacío.
---

## ParticipanteOperacion (@Embeddable)
Reemplaza el patrón "dual null" de `cliente nullable + eventual nullable` en `Pago`, `Venta` y operaciones con espacio.

**Atributos:**

    - private TipoParticipante tipoParticipante;
    - private Long clienteId;                           ← null si es eventual
    - private DatosPersonaEventual eventual;            ← null si es cliente registrado

**Relaciones:**
- Ninguna directa. Referencia a `Cliente` por ID.

**Datos Importantes:**
- Constructor con factory methods estáticos:

      ParticipanteOperacion.deCliente(Long clienteId)
      ParticipanteOperacion.deEventual(DatosPersonaEventual datos)

- Constructor privado. Solo se instancia vía factory methods.
- Elimina el problema de dos campos mutuamente excluyentes en `Pago`, `Venta`, `Reserva` y `Evento`.
- `TipoParticipante`: `CLIENTE_REGISTRADO`, `PERSONA_EVENTUAL`.
- Validación en factory methods: exactamente uno de los dos campos debe ser no-null. El otro es null por contrato.
- Implementar `equals` y `hashCode` por valor.
---

## OrigenOperacion (@Embeddable)
Value Object de trazabilidad que referencia la operación que originó otra acción del sistema. Inmutable.

**Atributos:**

    - private final TipoOrigenOperacion tipoOrigen;
    - private final Long idOrigen;
    - private final String descripcion;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Inmutable. Una vez creado el origen, no cambia.
- Provee trazabilidad polimórfica sin acoplar clases transversales a todas las entidades.
- Factory method recomendado: `OrigenOperacion.de(TipoOrigenOperacion tipo, Long id, String desc)`.
- `tipoOrigen` y `idOrigen` no pueden ser null. `descripcion` es opcional.
---

## ReferenciaCobrable (@Embeddable)
Referencia polimórfica a cualquier entidad que implemente `Cobrable`. Permite que `AplicacionPago` no esté acoplada
exclusivamente a `Cuota`.

**Atributos:**

    - private final TipoCobrable tipoCobrable;
    - private final Long cobradoId;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Inmutable.
- `TipoCobrable`: `CUOTA`, `RESERVA`, `EVENTO`, `VENTA`.
- Permite aplicar pagos a cualquier tipo de deuda del sistema.
- Factory methods: `ReferenciaCobrable.paraCuota(Long id)`, `.paraReserva(Long id)`,`.paraEvento(Long id)`, 
  `.paraVenta(Long id)`.
- Si el negocio garantiza que pagos SOLO aplican a cuotas, puede documentarse como extensión futura y usar `Cuota cuota`
  directamente.
---

## HorarioConcreto (Extiende de RangoHorario) (@Embeddable)
Clase reutilizable para agrupar datos de fecha y horario concreto.

**Atributos:**

    - private LocalDate fecha;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Modela inicio y fin de una ocupación en una fecha específica.
- Validar que `horaDeFin` sea posterior a `horaDeInicio`.
- Método `solapaCon(HorarioConcreto otro)`: devuelve true si `this` y `otro` tienen la misma `fecha` y sus rangos horarios
  se superponen.
---

## HorarioRecurrente (Extiende de RangoHorario) (@Embeddable)
Clase reutilizable para representar un patrón semanal de horario.

**Atributos:**

    - private DayOfWeek diaSemana;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- Un servicio usa este patrón semanal para generar `OcupacionEspacio` en fechas concretas.
- Validar en constructor: `diaSemana` no nulo, `horaDeFin` posterior a `horaDeInicio`.
- No representa una ocupación real de fecha concreta.
---

## UbicacionBackup (@Embeddable)
Referencia a la ubicación física o remota donde se almacena un backup. Inmutable.

**Atributos:**

    - private final TipoUbicacionBackup tipo;   ← LOCAL, REMOTO
    - private final String ruta;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- `ruta` no puede ser nula ni vacía.
- Para tipo `LOCAL`: ruta es una ruta del sistema de archivos.
- Para tipo `REMOTO`: ruta es una URI (Ej.: `s3://bucket/path`, `ftp: //host/path`, `gdrive://folder`).
- Inmutable. Una vez creada la ubicación, no se modifica.
- Factory methods recomendados: `UbicacionBackup.local(String ruta)`, `UbicacionBackup.remoto(String uri)`.
---

## RecargoPorMora (@Embeddable)
Value Object para modelar el recargo aplicable por mora en cuotas vencidas. Inmutable.

**Atributos:**

    - private final TipoRecargo tipo;    
    - private final BigDecimal valor;

**Relaciones:**
- Ninguna.

**Datos Importantes:**
- `TipoRecargo`: `PORCENTAJE`, `MONTO_FIJO`.
- Si `tipo = PORCENTAJE`: `valor` representa el porcentaje (Ej.: `5.00` = 5%).
- Si `tipo = MONTO_FIJO`: `valor` representa un importe en la moneda base del sistema.
- `valor` no puede ser negativo.
- Método `calcularRecargo(Dinero importeBase)`: devuelve el `Dinero` de recargo según el tipo.
---
#
#
#
#
#
# 3. ENTIDADES DEL DOMINIO
#

## Sesion (Extiende de AuditoriaEntidad)
Registra sesiones o tokens activos/inactivos. Ayuda a cerrar sesiones, auditar accesos y controlar estado.

**Atributos:**

    - private Long usuarioId;
    - private String token;
    - private EstadoSesion estado;
    - private LocalDateTime inicioSesion;
    - private LocalDateTime finSesion;

**Relaciones:**
- Ninguna directa. La relación es unidireccional: `Sesion → usuarioId`. `Usuario` NO tiene referencia a `Sesion`.

**Datos Importantes:**
- `token` es necesario para invalidar sesiones activas.
- Se pueden buscar sesiones activas de un usuario mediante `SesionRepositorio.buscarActivas(usuarioId)`.
- No debe contener contraseñas ni datos sensibles más allá del token.
- `finSesion` es null mientras la sesión esté `ACTIVA`.
---

## Usuario (Extiende de AuditoriaEntidad)
Representa a una persona que usa el sistema.

**Atributos:**

    - private NombrePersona nombre;
    - private String username;
    - private String passwordHash;
    - private EstadoUsuario estado;
    - private Long rolId;

**Relaciones:**
- No tiene referencia directa a `Sesion`. Las sesiones se consultan por `SesionRepositorio.buscarActivas(id)`.
- No tiene referencia directa a `Rol`. Se referencia por `rolId` para evitar cargas innecesarias.

**Datos Importantes:**
- `passwordHash` se obtiene siempre a través del puerto `PasswordHasher`. Nunca almacenar ni procesar la contraseña en
  texto plano fuera del puerto.
- `username` debe ser único en el sistema. Restricción de base de datos y validación en servicio.
- No permitir desactivar el último `ADMINISTRADOR` activo (regla de negocio a validar en servicio).
- Cada operación crítica del sistema asocia el `id` del usuario responsable vía `creadoPorId` / `actualizadoPorId` heredados
  de `AuditoriaEntidad`.
---

## Rol (Extiende de AuditoriaEntidad)
Agrupa permisos funcionales del sistema.

**Atributos:**

    - private String nombre;
    - private String descripcion;
    - private List<Permiso> permisos;

**Relaciones:**
- Contiene `List< Permiso >` como parte del Aggregate `Rol`.

**Datos Importantes:**
- Perfiles típicos: `ADMINISTRADOR`, `ENCARGADO`, `EMPLEADO`, `CONSULTA`.
- El rol reduce repetición. Si un encargado puede anular ventas, pero no pagos, se cambia la asignación sin tocar código.
---

## Permiso (Extiende de AuditoriaEntidad)
Acción concreta habilitante dentro del sistema.

**Atributos:**

    - private PermisoID habilita;
    - private ModuloSistema modulo;
    - private String descripcion;

**Relaciones:**
- Pertenece al Aggregate `Rol`.

**Datos Importantes:**
- Separar Rol de Permiso permite flexibilidad máxima: Si mañana un encargado puede anular ventas, pero no pagos, se modifica
  la asignación de permisos al rol sin tocar código.
---

## AuditoriaOperacion (Extiende de AuditoriaEntidad)
Registro histórico de una acción importante realizada dentro del sistema. Sirve para guardar evidencia de operaciones
críticas.

**Atributos:**

    - private Long usuarioEjecutorId;
    - private ModuloSistema modulo;
    - private TipoOperacionAuditoria tipoOperacion;
    - private ResultadoAuditoria resultado;
    - private TipoEntidadAuditada entidadAfectada;       
    - private Long idEntidadAfectada;                    
    - private String valorAnterior;                      
    - private String valorNuevo;                         
    - private String motivo;                             ← si corresponde (dato mínimo de auditoría)
    - private OrigenOperacion origenOperacion;
    - private String detalle;

    (creadoEn heredado de AuditoriaEntidad actúa como fechaHora del evento).

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- `creadoEn` de `AuditoriaEntidad` registra el momento exacto del evento.
- `usuarioEjecutorId` puede ser null para procesos automáticos del sistema.
- Se persiste vía el puerto `AuditoriaPort` (escritura). La consulta filtrada (pantalla "Consulta de auditoría" del
  Flujo 12: filtros fecha desde/hasta, usuario y acción) se resuelve vía `AuditoriaConsultaPort` (puerto de lectura
  separado, ver más abajo).
- `usuarioEjecutorId` = Usuario; `creadoEn` = Fecha/hora; 
- `tipoOperacion` = Acción; `entidadAfectada` + `idEntidadAfectada` = Entidad afectada + ID; 
- `valorAnterior` / `valorNuevo` = Datos anteriores / nuevos; 
- `motivo` es un campo dedicado (distinto de `detalle`, texto técnico libre) porque el Flujo 12 lo lista como dato mínimo 
  propio y el Flujo 11 exige registrar el motivo de cada anulación.
- Esta entidad es de solo escritura (append-only). Nunca se modifica un registro de auditoría existente.
---

## Cliente (Extiende de Persona)
Representa a un niño o persona registrada como cliente permanente del complejo.

**Atributos:**

    - private EstadoCliente estado;
    - private Genero genero;
    - private LocalDate fechaNacimiento;
    - private LocalDate fechaAlta;
    - private LocalDate fechaBaja;

**Relaciones:**
- `SaldoAFavorCliente` (entidad separada, 1:1 con Cliente) — ver definición más abajo.
- `ResponsableDelCliente` (relación N:M via tabla intermedia).
- `Inscripcion` (consultada vía `InscripcionRepositorio`, no como lista embebida).

**Datos Importantes:**
- El cliente NO tiene `List< Inscripcion >` ni `List< Cuota >` como atributos. Se consultan desde sus repositorios para
  evitar listas bidireccionales pesadas.
- El cliente NO tiene `Dinero saldoAFavor`. El saldo vive en `SaldoAFavorCliente`.
- Los clientes no se eliminan; se marcan como `INACTIVO`.
- Si el cliente es mayor de edad, los responsables son opcionales.
- `fechaNacimiento` no puede ser una fecha futura. Validar en constructor / factory method.
- Un `Cliente` puede crearse y persistirse válidamente sin ninguna `Inscripcion` asociada (alta de cliente sin inscripción
  inicial). No es un estado inconsistente: al no existir relación de contención entre `Cliente` e `Inscripcion` (se consultan
  por repositorio, no como lista embebida), el aggregate `Cliente` queda completo con solo sus datos personales, responsable(s)
  y saldo a favor en cero. El cliente queda disponible para inscribirse más adelante; mientras no tenga inscripción activa, no
  participa de la generación de cuotas.
- Método `edad(LocalDate fechaReferencia)`: calcula la edad en años cumplidos a partir de `fechaNacimiento`. Se usa para
  validar restricciones de `edadMinima` / `edadMaxima` de `Actividad` al inscribir. Recibe la fecha de referencia (en lugar
  de usar `LocalDate.now()` internamente) para mantener la entidad testeable y determinística.
- Factory methods:
    - `Cliente.conDocumento(NombrePersona, DocumentoIdentidad, Contacto, ...)` → documento obligatorio.
    - `Cliente.sinDocumento(NombrePersona, Contacto, ...)` → solo disponible si `ConfiguracionSistema.preferencias.permitirRegistroSinDNI = true`. 
       La validación de esta condición es responsabilidad del servicio de aplicación.
---

## SaldoAFavorCliente (Extiende de AuditoriaEntidad)
Entidad que representa el crédito disponible de un cliente en el sistema.

**Atributos:**

    - private Long clienteId;              
    - private Dinero montoDisponible;

**Relaciones:**
- Ninguna directa. Vinculada a `Cliente` por `clienteId`.

**Datos Importantes:**
- Entidad con identidad propia. Se crea con `montoDisponible = Dinero.CERO` al registrar un cliente.
- No es un campo `Dinero` dentro de `Cliente`. Tiene su propio ciclo de vida y auditoría.
- Solo se modifica a través de `MovimientoSaldoCliente`.
- Regla de negocio: `montoDisponible` nunca puede ser negativo.
- Se consulta vía `SaldoAFavorClienteRepositorio.buscarPorCliente(clienteId)`.
- **Decisión de diseño respecto al flujo de negocío:** el flujo de alta de clientes solicita un atributo `saldoAFavorCuotas`
  dentro de `Cliente`. Se decide deliberadamente NO modelarlo así, sino como esta entidad separada 1:1, para evitar mezclar
  estado financiero mutable de alta concurrencia con el aggregate raíz `Cliente`, evitar conflictos de `@Version` entre
  actualizaciones de datos personales y de saldo, y mantener auditoría propia del saldo vía `MovimientoSaldoCliente`. El
  comportamiento funcional exigido por el flujo (cliente inicia con saldo cero, se consulta desde la ficha del cliente) queda
  igualmente garantizado.
---

## Responsable (Extiende de Persona)
Adulto responsable de uno o más clientes.

**Atributos:**

    - (No tiene nuevos atributos. Todos heredados de Persona.)

**Relaciones:**
- La relación con clientes se gestiona en `ResponsableDelCliente`.

**Datos Importantes:**
- Él `parentesco` se exige en el vínculo `ResponsableDelCliente`, no en esta entidad.
- `DocumentoIdentidad` es **opcional** para `Responsable`. Si se informa, debe cumplir el formato normalizado de 
  `DocumentoIdentidad`. No se exige DNI para poder asociar un responsable a un cliente.
- Como `Persona` modela `documento` como obligatorio, `Responsable` relaja esa restricción con factory methods propios
  (mismo patrón que `Cliente`), evitando que el constructor heredado de `Persona` exija un documento que el Flujo 1 declara
  opcional:
    - `Responsable.conDocumento(NombrePersona, DocumentoIdentidad, Contacto, ...)` → con documento.
    - `Responsable.sinDocumento(NombrePersona, Contacto, ...)` → sin documento.
  Ambos validan la precondición de mayoría de edad y los campos mínimos (`nombre`, `apellido`, `telefono`).
- `Responsable` es mayor de edad: precondición más estricta que `Persona`, validada en su constructor / factory method.
- No usar en polimorfismo donde se espera `Persona` sin restricción de edad.
- La relación con clientes se gestiona en `ResponsableDelCliente`.
---

## ResponsableDelCliente (Extiende de AuditoriaEntidad)
Vínculo entre un cliente y un responsable, con parentesco y prioridad de contacto.

**Atributos:**

    - private Long clienteId;                   
    - private Long responsableId;               
    - private Parentesco parentesco;
    - private boolean responsablePrincipal;
    - private EstadoRelacion estado;
    - private String observaciones;

**Relaciones:**
- Ninguna directa (referencias por ID).

**Datos Importantes:**
- **Nota de nomenclatura:** esta entidad corresponde a lo que el flujo de negocios (Flujo 1 - Alta de clientes e inscripciones)
  denomina `ClienteResponsable`. Se renombra a `ResponsableDelCliente` por claridad semántica de la relación (el responsable
  "pertenece a" / "es responsable de" el cliente). No hay diferencia funcional.
- Permite relaciones N:M entre clientes y responsables.
- `EstadoRelacion` evita borrado físico de vínculos.
- Restricción de negocio: solo puede existir un `responsablePrincipal = true` activo por cliente. Validar en servicio de
  aplicación.
---

## EspacioFisico (Extiende de AuditoriaEntidad)
Lugar físico que se ocupa: cancha, salón, quincho, gimnasio, etc.

**Atributos:**

    - private String nombre;
    - private TipoEspacio tipo;
    - private String descripcion;               
    - private Integer capacidad;                  
    - private EstadoRegistro estado;
    - private String observaciones;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Solo espacios con `estado = ACTIVO` pueden usarse en reservas, eventos y actividades.
- No borrar espacios. Marcar como `INACTIVO`.
- Espacios iniciales: cancha fútbol 5, sala taekwondo y salón infantil.
- El negocio administra espacios limitados. Separar espacio de actividad permite que varias actividades usen un mismo 
  lugar sin duplicar disponibilidad.
- **`descripcion` y `capacidad` (Flujo 9):** el flujo los declara como datos mínimos del espacio y los expone en el
  formulario de alta/edición. `descripcion` es texto libre descriptivo; `capacidad` es opcional (`null` cuando no aplica). 
  `observaciones` se conserva como nota operativa separada de la descripción funcional, ya que el formulario del Flujo 9 
  lista ambos campos por separado.
- No puede existir otro espacio `ACTIVO` con el mismo `nombre`. Validar en servicio y, para blindar concurrencia, índice 
  único parcial sobre `nombre` filtrado por `estado = ACTIVO`.
---

## Actividad (Extiende de AuditoriaEntidad)
Representa una actividad o servicio del complejo como escuela de fútbol, taekwondo, alquiler de fútbol 5, educación física
escolar, cumpleaños deportivos y salon infantil.

**Atributos:**

    - private String nombre;
    - private TipoActividad tipo;
    - private Long espacioFisicoId;                ← null si la actividad no ocupa un espacio fijo (Flujo 9: "si corresponde")
    - private Integer edadMinima; 
    - private Integer edadMaxima;
    - private boolean permiteInscripcion;
    - private boolean generaCuotaMensual;
    - private Dinero precioPorDefecto;              ← precio base sugerido de la actividad (opcional)
    - private EstadoActividad estado;
    - private String descripcion;

**Relaciones:**
- Ninguna directa. Referencia a `EspacioFisico` por `espacioFisicoId`.

**Datos Importantes:**
- Sin listas bidireccionales. Inscripciones, precios y horarios se consultan desde sus repositorios.
- Invariante: si `permiteInscripcion = false`, entonces `generaCuotaMensual` debe ser `false`. Esta invariante se valida
  en el constructor / factory method de la entidad.
- No todas las actividades generan cuota mensual.
- `precioPorDefecto` es opcional. Si está definido, se usa como "precio sugerido" en la pantalla de generación mensual
  cuando no existe un `PrecioMensualActividad` previo para el período. Si no está definido, el precio sugerido se obtiene
  del último `PrecioMensualActividad` vigente de la actividad.
---

## PrecioMensualActividad (Extiende de AuditoriaEntidad)
Representa el precio de una actividad en un periodo determinado.

**Atributos:**

    - private Long actividadId;
    - private PeriodoMensual periodoDesde;
    - private PeriodoMensual periodoHasta;      ← null = vigente hasta nuevo precio
    - private Dinero importe;
    - private boolean activo;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- `periodoHasta` puede ser null (precio vigente indefinidamente hasta que se cree uno nuevo).
- No sobreescribir precios usados por cuotas ya generadas. Si ya existe una `GeneracionCuota` confirmada para el período,
  el `PrecioMensualActividad` vigente para ese período no puede modificarse libremente desde el flujo de generación mensual.
  Solo se puede usar (sin modificar) en el sub-flujo de generación de cuotas faltantes.
- **Modelo por rango vs. por período (decisión superadora):** Aquí se modela como rango (`periodoDesde` / `periodoHasta`)
  para no duplicar filas cuando el precio se mantiene varios meses; el precio aplicable a un período concreto se resuelve
  con `buscarVigente(actividadId, periodo)`. El requisito de "precio histórico por mes" queda igualmente garantizado porque,
  además, cada `Cuota` congela su propio `importeTotal` al generarse: cambiar un precio futuro nunca altera cuotas de meses
  ya generados.
- Debe existir precio vigente mayor a cero para generar cuota.
- **Lógica del precio sugerido en pantalla:** cuando el administrador accede a la pantalla de generación mensual, el sistema
  sugiere un precio para cada actividad usando el siguiente orden de prioridad:
    1. El `importe` del `PrecioMensualActividad` ya definido para ese período (si existe).
    2. El `importe` del último `PrecioMensualActividad` activo de la actividad.
    3. El `precioPorDefecto` de `Actividad` (si el anterior no existe).
  El administrador puede aceptar o modificar el precio sugerido antes de confirmar la generación.
---

## Inscripcion (Extiende de AuditoriaEntidad)
Representa que un cliente está anotado en una actividad.

**Atributos:**

    - private Long clienteId;
    - private Long actividadId;
    - private LocalDate fechaAlta;
    - private LocalDate fechaBaja;
    - private Integer diaVencimiento;            ← día del mes (1-28) para calcular Cuota.fechaVencimiento en cada generación
    - private Dinero precioMensualPactado;       ← precio acordado al alta (opcional; null = usar PrecioMensualActividad)
    - private EstadoInscripcion estado;
    - private String observaciones;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- No permitir duplicado: un cliente no puede tener dos inscripciones `ACTIVA` en la misma actividad. Restricción validada
  en servicio de aplicación y `InscripcionRepositorio.existeInscripcionActiva(clienteId, actividadId)`. Para blindar esta
  regla bajo concurrencia (igual criterio que la restricción única de `Cuota`), se recomienda un índice único parcial a
  nivel de base de datos sobre `(clienteId, actividadId)` filtrado por `estado = ACTIVA`, de forma que dos altas simultáneas
  no puedan generar el duplicado que la validación en servicio, por sí sola, no alcanza a prevenir.
- Respetar restricciones de edad de la actividad al momento del alta.
- `fechaBaja` es null mientras la inscripción esté activa.
- Solo inscripciones `ACTIVA` generan cuotas.
- `precioMensualPactado`: precio mensual acordado al momento del alta de la inscripción. Se captura en este flujo y
  queda persistido aquí (no solo en `PrecioMensualActividad`) porque el flujo de negocios permite cargarlo manualmente al
  inscribir, pudiendo diferir de la tabla de precios general (Ej.: descuentos particulares, becas). El proceso de generación
  mensual de cuotas (`GeneracionCuota`) debe priorizar este valor sobre `PrecioMensualActividad.buscarVigente(...)`cuando
  esté presente.
- `diaVencimiento`: día del mes (1-28, ver `ConfiguracionCuotas.diaVencimiento` para el valor por defecto a nivel
  sistema) pactado para esta inscripción en particular. Permite que cada inscripción tenga su propio vencimiento,
  independiente del valor general de `ConfiguracionCuotas`. Se usa para calcular `Cuota.fechaVencimiento` en cada
  generación mensual.
- Validar en constructor / factory method: `diaVencimiento` entre 1 y 28 (inclusive); `precioMensualPactado`, si está
  presente, mayor a `Dinero.CERO` (puede ser null para usar el `PrecioMensualActividad` vigente).
- Estado inicial: una inscripción nueva se crea en `EstadoInscripcion.ACTIVA` por defecto. El estado inicial no es 
  editable en el alta común; solo se modifica desde un flujo especial o con permiso avanzado.
- `fechaAlta` (fecha de inicio) es obligatoria y no puede estar vacía. Una fecha de inicio demasiado anterior o incoherente 
  requiere confirmación explícita del administrador a nivel de servicio.
---

## Cuota (Extiende de OperacionCobrableBase e implementa Cobrable y ReceptorDePago)
Obligación mensual generada por una inscripción.

**Atributos:**

    - private Long clienteId;
    - private Long inscripcionId;
    - private Long actividadId;                  
    - private Long generacionCuotaId;            
    - private PeriodoMensual periodo;
    - private LocalDate fechaVencimiento;
    - private EstadoCuota estadoCuota;
    - private EstadoVencimiento estadoVencimiento;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- `clienteId` es desnormalización consciente para facilitar queries de deuda por cliente sin join a `Inscripcion`.
- **Restricción compuesta de unicidad:** `clienteId + actividadId + periodo`, reglas "No se podrá generar una cuota 
  duplicada para el mismo cliente, actividad y período" y "Deberá existir una restricción única lógica cliente + actividad + período". 
  Garantizada por la base de datos y validada por `CuotaRepositorio.existeParaPeriodo(clienteId, actividadId, periodo)`.
  **Nota de diseño:** `inscripcionId` se conserva como referencia precisa a la inscripción que originó la cuota (trazabilidad
  y cálculo de vencimiento), pero NO forma parte de la clave de unicidad. Usar `inscripcionId` en la clave dejaría un hueco
  en un caso de reinscripción dentro del mismo período: si una inscripción "A" genera la cuota del mes, luego se finaliza 
  y se crea una inscripción "B" `ACTIVA` para el mismo cliente y actividad en ese mismo período, una generación de cuotas 
  faltantes evaluaría `existeParaPeriodo` contra "B" —no contra "A"— y crearía una segunda cuota para el mismo cliente, 
  actividad y período, violando la regla del flujo. La clave `cliente + actividad + período` cierra ese hueco y es la 
  exigida explícitamente por el negocio. `actividadId` es desnormalización consciente (igual criterio que `clienteId`): 
  Evita un join a `Inscripcion` para validar el duplicado y soporta el índice único a nivel de base de datos.
- `importeTotal` y `importePagado` heredados de `OperacionCobrableBase`. **`importePagado` representa el importe total
  cubierto de la cuota = pago real (vía `AplicacionPago`) + saldo a favor aplicado (vía `AplicacionSaldoFavorCuota`).**
  Por eso `getSaldoPendiente()` llega a cero —y la cuota queda `PAGADA`— tanto si se cubrió con dinero real como con saldo
  a favor o con ambos. El desglose real vs. saldo a favor nunca se lee de `importePagado`: se reconstruye desde 
  `AplicacionPago` y `AplicacionSaldoFavorCuota`.
- `generacionCuotaId` es null para cuotas creadas fuera del proceso masivo (Ej.: alta manual). Permite trazar el origen
  de cada cuota al proceso de generación que la produjo.
- **Dos dimensiones de estado independientes (requisito explícito del Flujo 3):** el flujo exige separar estado de pago
  y estado de vencimiento para evitar ambigüedad. Una cuota `PARCIAL` y vencida debe mostrarse simultáneamente como
  `PARCIAL` (estado de pago) y `VENCIDA` (estado de vencimiento), algo imposible de representar con un único campo. Por
  este motivo se declaran dos campos separados:
    - `estadoCuota` (estado de pago): `PENDIENTE`, `PARCIAL`, `PAGADA`, `ANULADA`.
    - `estadoVencimiento` (estado de vencimiento): `AL_DIA`, `VENCIDA`, `SIN_DEUDA`.
- **Fuente de verdad del estado de pago:** `estadoCuota` es el campo autorizado para el estado de ciclo de vida de
  pago de la cuota. `estadoCobro` heredado de `OperacionCobrableBase` es la proyección del mismo hecho para los contratos
  `Cobrable` / `ReceptorDePago`. **No se escriben de forma independiente:** ambos derivan del saldo pendiente y se
  recalculan juntos en la misma operación. La cuota expone **tres** —y solo tres— puntos de entrada para mutar su saldo,
  ambos atómicos y ambos responsables de derivar de forma consistente `importePagado`, `saldoPendiente`, `estadoCuota`,
  `estadoCobro` y `estadoVencimiento`, evitando el riesgo de desincronización por doble escritor:
    - `registrarAplicacionPago(AplicacionPago)` (contrato `ReceptorDePago`, override en `Cuota`): único punto de entrada
      para el **pago real**.
    - `registrarAplicacionSaldoFavor(Dinero monto)`: único punto de entrada para la **aplicación de saldo a favor**. 
      Necesario porque `importePagado` también suma el saldo a favor aplicado, pero ese camino NO pasa por una `AplicacionPago`. 
      Sin este método la cuota no podría pasar a `PARCIAL`/`PAGADA` por saldo a favor manteniendo sus estados sincronizados. 
      Lo invoca la implementación de `PoliticaDeAplicacionSaldoFavor.aplicarSaldo(...)` y, además del cambio de estado, 
      deja registro vía `AplicacionSaldoFavorCuota` y `MovimientoSaldoCliente`.
    - `revertirAplicacionPago(AplicacionPago)` / `revertirAplicacionSaldoFavor(Dinero monto)`: punto de entrada para la
      **anulación** de un pago o de una aplicación de saldo. Resta de `importePagado` el monto que se había aplicado y 
      vuelve a derivar `estadoCuota`, `estadoCobro` y `estadoVencimiento` desde el nuevo saldo pendiente. Una cuota que 
      estaba `PAGADA` puede volver a `PENDIENTE`/`PARCIAL` y, si su `fechaVencimiento` ya pasó, `estadoVencimiento` vuelve 
      a `VENCIDA`. Lo invoca `CuotaHandler` al consumir `PagoAnulado`.
  - Mapeo `estadoCuota` → `estadoCobro`:
    - `PAGADA` → `SALDADO`
    - `PARCIAL` → `PARCIAL`
    - `PENDIENTE` → `PENDIENTE`
    - `ANULADA` → `ANULADO`
- **Sincronización de `estadoVencimiento`:** al crear la cuota, se inicializa en `AL_DIA`. Un servicio programado lo
  transiciona a `VENCIDA` cuando `fechaVencimiento < LocalDate.now()` y `estadoCuota` es `PENDIENTE` o `PARCIAL`. Cuando
  `estadoCuota` pasa a `PAGADA` **o `ANULADA`** (en ambos casos la cuota deja de tener saldo exigible), `estadoVencimiento`
  pasa a `SIN_DEUDA`. La derivación ocurre en la misma operación que cambia `estadoCuota` (registro de pago, aplicación de
  saldo a favor o anulación), no como escritura suelta posterior.
- **Estado `PAGADA` (CORRECCIÓN de diseño):** una cuota queda `PAGADA` cuando su saldo pendiente llega a cero, sin
  importar si se cubrió con un pago real, con saldo a favor o con una combinación de ambos. El origen del pago (real
  vs. saldo a favor) siempre se reconstruye desde `AplicacionPago` y `AplicacionSaldoFavorCuota`.
---

## GeneracionCuota (Extiende de AuditoriaEntidad)
Registro de la ejecución de un proceso masivo de generación de cuotas para un período.

**Nota de nomenclatura:** Esta entidad corresponde al concepto `GeneracionCuotasMensuales` definido en el Flujo 2. Se
abrevia a `GeneracionCuota` por concisión y porque el contexto mensual está implícito en el dominio. Toda referencia
a `GeneracionCuotasMensuales` en el flujo de negocios apunta a esta entidad.

**Atributos:**

    - private PeriodoMensual periodo;
    - private Long usuarioConfirmacionId;           ← ID del usuario que confirmó la generación (null hasta confirmación)
    - private LocalDateTime fechaConfirmacion;      ← null hasta que se confirma
    - private Integer cantidadGeneradas;
    - private Integer cantidadOmitidas;
    - private Integer cantidadPendientes;           
    - private Integer cantidadParciales;            
    - private Integer cantidadPagadas;              
    - private Dinero totalBruto;                    
    - private Dinero totalSaldoAplicado;            
    - private Dinero totalPendiente;                
    - private boolean esComplementaria;             ← true si es una generación de cuotas faltantes
    - private EstadoGeneracionCuota estado;

**Relaciones:**
- `List< DetalleGeneracionCuota >` (entidades internas del Aggregate, no se navegan como lista en la entidad raíz; se
  consultan por `DetalleGeneracionCuotaRepositorio.buscarPorGeneracion(generacionId)`).

**Datos Importantes:**
- `creadoEn` heredado de `AuditoriaEntidad` actúa como `fechaHoraInicio` del proceso (momento en que se inició la
  transacción). `creadoPorId` heredado actúa como identificador del usuario que inició el proceso.
- `usuarioConfirmacionId` registra específicamente quién confirmó la generación (puede diferir de quien la inició si
  el sistema permite que un usuario distinto confirme). Se puebla al transicionar a `CONFIRMADA`.
- `fechaConfirmacion` se registra cuando el estado transiciona a `CONFIRMADA`.
- Permite saber si el proceso ya se ejecutó para un período.
- Restricción única lógica sobre `periodo` para la generación normal (índice único parcial recomendado: sobre `periodo`
  filtrado por `esComplementaria = false` y `estado = CONFIRMADA`), de modo que no puedan existir dos generaciones normales
  confirmadas del mismo período. Las generaciones complementarias (`esComplementaria = true`) pueden coexistir con la
  generación confirmada del mismo período (sub-flujo de cuotas faltantes).
- `EstadoGeneracionCuota`: `PREVISUALIZACION`, `EN_PROCESO`, `CONFIRMADA`, `FALLIDA`, `ANULADA`.
    - `EN_PROCESO`: se asigna al iniciar la transacción atómica. Permite detectar procesos colgados y bloquear ejecuciones
      concurrentes del mismo período.
    - `ANULADA`: permite invalidar una generación confirmada en casos excepcionales autorizados.
- La previsualización genera una `GeneracionCuota` en estado `PREVISUALIZACION` sin persistir cuotas. La confirmación la
  transiciona a `CONFIRMADA` y persiste las cuotas.
- `totalBruto`, `totalSaldoAplicado` y `totalPendiente` son `Dinero.CERO` hasta la confirmación y se calculan durante el
  proceso de generación.
- `cantidadPendientes`, `cantidadParciales` y `cantidadPagadas` desglosan las cuotas generadas por estado de pago final,
  requisito explícito del resumen final (Flujo 2 paso 84). Se almacenan en la generación (no solo se derivan en pantalla) 
  para que el resumen quede persistido y auditable junto al resto de los totales. Invariante: 
  `cantidadPendientes + cantidadParciales + cantidadPagadas = cantidadGeneradas`.
---

## DetalleGeneracionCuota (Entidad interna del Aggregate GeneracionCuota)
Representa el detalle de una cuota generada u omitida durante una generación mensual.

**Atributos:**

    - private Long generacionId;
    - private Long clienteId;                        
    - private Long actividadId;                      
    - private Long inscripcionId;
    - private Long cuotaGeneradaId;                  ← ID (null si fue omitida)
    - private ResultadoGeneracionCuota resultado;
    - private String motivoOmision;
    - private Dinero importePrevisto;                
    - private Dinero saldoAplicado;                  
    - private Dinero saldoPendienteFinal;            

**Relaciones:**
- Ninguna directa. Pertenece al Aggregate `GeneracionCuota`.

**Datos Importantes:**
- Permite conservar el resumen detallado de una generación mensual.
- Ayuda a explicar cuotas omitidas por duplicado, sin precio vigente, inscripción inactiva, inicio posterior al período
  o actividad que no genera cuota mensual.
- `motivoOmision` es null cuando `resultado = GENERADA`.
- `importePrevisto`, `saldoAplicado` y `saldoPendienteFinal` son `Dinero.CERO` cuando `resultado != GENERADA`.
- `clienteId` y `actividadId` son desnormalizaciones conscientes: permiten auditar directamente sin joins a `Inscripcion`.
---

## AplicacionSaldoFavorCuota (Extiende de AuditoriaEntidad)
Registro histórico de la aplicación de saldo a favor sobre una cuota determinada. Provee trazabilidad individual de cada
descuento de saldo, independientemente del movimiento de saldo que lo originó.

**Atributos:**

    - private Long clienteId;
    - private Long cuotaId;
    - private Dinero montoAplicado;
    - private Long movimientoSaldoId;               
    - private Long usuarioOProcesoId;               ← null si fue proceso automático del sistema
    - private String motivo;                        

    (creadoEn heredado de AuditoriaEntidad actúa como fechaHoraAplicacion).

**Relaciones:**
- Ninguna directa. Vinculada a `Cuota` por `cuotaId` y a `MovimientoSaldoCliente` por `movimientoSaldoId`.

**Datos Importantes:**
- Registro append-only. Una vez creada, no se modifica.
- `montoAplicado` debe ser mayor a `Dinero.CERO` y no puede superar el saldo pendiente de la cuota al momento de la
  aplicación.
- **No genera `MovimientoCaja`**. El dinero ya fue registrado en caja cuando el cliente realizó el pago original que
  generó el saldo a favor.
- `movimientoSaldoId` permite navegar desde esta aplicación hacia el movimiento de saldo que la respaldó, garantizando
  trazabilidad bidireccional.
- Se consulta vía `AplicacionSaldoFavorCuotaRepositorio.buscarPorCuota(cuotaId)` para ver el historial completo de
  aplicaciones de una cuota.
---

## MovimientoSaldoCliente (Extiende de AuditoriaEntidad)
Historial de aumentos y aplicaciones de saldo a favor. Cada registro representa un evento que modifica el`montoDisponible` 
de `SaldoAFavorCliente`, con trazabilidad del antes y después.

**Atributos:**

    - private Long clienteId;
    - private TipoMovimientoSaldo tipoMovimientoSaldo;
    - private Dinero monto;
    - private Dinero saldoAnterior;
    - private Dinero saldoPosterior;
    - private OrigenOperacion origenOperacion;
    - private String observacion;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- El saldo a favor no impacta caja porque no entra dinero nuevo cuando se consume saldo a favor.
- Registro append-only. No se modifica un movimiento de saldo existente.
- `TipoMovimientoSaldo` determina el origen del movimiento:
  - `GENERACION_SALDO_A_FAVOR`: pago que supera la deuda del cliente → acredita saldo.
  - `APLICACION_SALDO_A_CUOTA`: saldo consumido para cubrir una cuota → debita saldo.
  - `ANULACION_SALDO_A_FAVOR`: anulación de un pago que había generado saldo a favor → debita el saldo acreditado.
  - `AJUSTE_MANUAL`: corrección autorizada por administrador → puede acreditar o debitar.
- `saldoPosterior`: cuando tipo es `GENERACION_SALDO_A_FAVOR` o `AJUSTE_MANUAL` positivo: `saldoAnterior + monto`. Cuando
  tipo es `APLICACION_SALDO_A_CUOTA`, `ANULACION_SALDO_A_FAVOR` o `AJUSTE_MANUAL` negativo: `saldoAnterior - monto`.
- `origenOperacion` apunta a la entidad que originó el movimiento: `Pago` en `GENERACION_SALDO_A_FAVOR` y en
  `ANULACION_SALDO_A_FAVOR`, `Cuota` en `APLICACION_SALDO_A_CUOTA`, nulo o descripción en `AJUSTE_MANUAL`.
---

## Pago (Extiende de OperacionBase)
Ingreso real de dinero recibido por el negocio.

**Atributos:**

    - private ParticipanteOperacion participante;
    - private Dinero totalRecibido;
    - private List<DetalleMetodoPago> metodosPago;
    - private List<AplicacionPago> aplicaciones;

    (fechaDeLaOperacion heredada de OperacionBase actúa como fechaPago).

**Relaciones:**
- Contiene `List< DetalleMetodoPago >` y `List< AplicacionPago >` como partes del Aggregate `Pago`.

**Datos Importantes:**
- `ParticipanteOperacion` reemplaza la dualidad `Cliente cliente + DatosPersonaEventual eventual`.
- Todo pago real genera `MovimientoCaja` (coordinado vía Domain Event `PagoRegistrado`).
- No borrar pagos: anular y compensar caja / saldo / cuotas con eventos compensatorios.
- Un método de pago por pago (simplificación documentada). La estructura `List< DetalleMetodoPago >` está preparada para
  pagos mixtos en versiones futuras.
- `Pago` **no declara un campo `estadoPago` propio**. `EstadoPago` (`ACTIVO`/`ANULADO`) es semánticamente idéntico a
  `EstadoOperacion` (`ACTIVA`/`ANULADA`) heredado de `OperacionBase`: representar el mismo hecho con dos campos crearía una
  única fuente de verdad duplicada, con riesgo de desincronización. Esto respeta la regla ya establecida en `OperacionBase`:
  las subclases no agregan un segundo campo de estado que solape con `estado`.
- La anulación de un pago se expresa exclusivamente como `estado = ANULADA`, igual que el resto de las subclases de
  `OperacionBase`.
- **Anulación:** la etiqueta "REGISTRADO" que muestra el flujo es la presentación de `estado = ACTIVA`. La verificación 
  "el pago está REGISTRADO" previa a anular equivale a comprobar `estado == ACTIVA`, lo que también impide anular dos veces 
  la misma operación. El motivo, usuario y fecha/hora de la anulación NO se guardan como campos de `Pago`: viven en la 
  `CancelacionOperacion` (`origenOperacion`=`PAGO`) y en el `MovimientoCaja` de tipo `ANULACION` (ver `PagoAnulado`).
---

## DetalleMetodoPago (Extiende de AuditoriaEntidad)
Forma concreta en que se recibió parte o todo un pago.

**Atributos:**

    - private Long pagoId;
    - private MetodoPago metodoPago;
    - private Dinero monto;
    - private String referenciaExterna;

**Relaciones:**
- Pertenece al Aggregate `Pago`. Ciclo de vida controlado por `Pago`.

**Datos Importantes:**
- `monto` debe ser mayor a `Dinero.CERO`.
- `referenciaExterna` es opcional. Permite registrar número de comprobante, código de transacción o referencia de
  transferencia para métodos no presenciales (`TRANSFERENCIA`, `MERCADO_PAGO`). Para `EFECTIVO` o `DEBITO`/`CREDITO` puede
  ser null.
- La suma de `monto` de todos los `DetalleMetodoPago` de un mismo `Pago` debe ser igual a `Pago.totalRecibido`.
---

## AplicacionPago (Entidad interna del Aggregate Pago, extiende de AuditoriaEntidad)
Parte de un pago aplicada a una deuda concreta.

**Atributos:**

    - private Long pagoId;
    - private ReferenciaCobrable destino;
    - private Dinero montoAplicado;
    - private String observacion;

    (creadoEn heredado de AuditoriaEntidad actúa como fechaAplicacion).
    (creadoPorId heredado de AuditoriaEntidad actúa como usuarioQueRegistro).

**Relaciones:**
- Pertenece al Aggregate `Pago`. Ciclo de vida controlado por `Pago`.

**Datos Importantes:**
- Extiende de `AuditoriaEntidad` para garantizar los campos `creadoEn` (fecha y hora de aplicación) y `creadoPorId`
  (usuario que registró la operación), ambos requeridos explícitamente por el Flujo 3 (paso 49).
- `ReferenciaCobrable` (Value Object con `TipoCobrable` + `cobradoId`) permite aplicar pagos a cualquier `Cobrable` del sistema.
- **Equivalencia de nomenclatura (Flujo 8):** el dato `AplicacionPagoCuota` que lista el Flujo 8 corresponde a esta
  entidad `AplicacionPago` con `destino.tipoCobrable = CUOTA`. Se generaliza el nombre porque el mismo mecanismo aplica
  pagos a `RESERVA`/`EVENTO`; el informe de cuotas y de pagos por cliente la consulta filtrando por `TipoCobrable.CUOTA`.
- En V1 se aplican pagos a `CUOTA` (Flujo 3) y a `RESERVA` / `EVENTO` (Flujos 5 y 6: seña, pago parcial o pago total).
  `VENTA` NO pasa por `Pago` en V1 (la venta se autosalda al registrarse, Flujo 4); `TipoCobrable.VENTA` queda preparado
  para un eventual "fiado" futuro.
- La suma de `montoAplicado` de todas las aplicaciones de un pago debe ser ≤ `Pago.totalRecibido`.
---

## ComprobanteOperacion (Extiende de AuditoriaEntidad)
Comprobante emitido por una operación económica.

**Atributos:**

    - private TipoComprobante tipoComprobante;
    - private String numero;                        ← (Ej:  REC-000001)
    - private OrigenOperacion origenOperacion;
    - private Dinero total;
    - private EstadoComprobante estado;

    (fechaEmision = creadoEn heredado).

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- La numeración se controla con `SecuenciaComprobante` de forma transaccional (`SELECT FOR UPDATE`).
- `numero` es único por `TipoComprobante`.
- Un comprobante anulado (`estado = ANULADO`) no puede reactivarse.
---

## CajaDiaria (Extiende de AuditoriaEntidad)
Jornada de caja de un día, con apertura, cierre y responsable.

**Atributos:**

    - private LocalDate fecha;
    - private LocalDateTime fechaHoraApertura;
    - private LocalDateTime fechaHoraCierre;
    - private Long usuarioAperturaId;
    - private Long usuarioCierreId;
    - private Dinero montoInicial;
    - private Dinero montoCierre;
    - private EstadoCajaDiaria estado;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- `usuarioAperturaId` / `usuarioCierreId` como ID's para evitar cargar objetos `Usuario` al consultar caja.
- `estado`: `ABIERTA`, `CERRADA`.
- Solo puede existir una `CajaDiaria` con `estado = ABIERTA` por vez. Validar en servicio.
- `fechaHoraCierre` y `usuarioCierreId` son null mientras la caja esté `ABIERTA`.
- `montoCierre` es null hasta el momento del cierre.
- **Alcance V1 (Flujo 7):** la version 1 NO realiza cierre formal de caja con arqueo. `montoInicial`, `montoCierre`,
  apertura y cierre quedan modelados como preparación futura, pero su gestion es opcional/automática. En V1 puede existir
  una `CajaDiaria` por fecha, creada automáticamente al registrar el primer `MovimientoCaja` del día, sin flujo manual de
  apertura/cierre.
---

## MovimientoCaja (Extiende de AuditoriaEntidad)
Entrada o salida real de dinero en caja.

**Atributos:**

    - private Long cajaDiariaId;
    - private TipoMovimientoCaja tipoMovimientoCaja;
    - private AreaComplejo area;                          
    - private String concepto;                            
    - private MetodoPago metodoPago;
    - private Dinero monto;
    - private ParticipanteOperacion participante;         
    - private OrigenOperacion origenOperacion;
    - private Long movimientoOriginalAnuladoId;           ← != null solo si tipoMovimientoCaja = ANULACION
    - private String motivoAnulacion;                     ← != null solo si tipoMovimientoCaja = ANULACION
    - private EstadoMovimientoCaja estado;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Separado de `Pago` porque no todo movimiento de caja es un pago (devoluciones, ajustes manuales).
- `concepto` es el texto de presentación de la grilla de caja (Ej.: "Pago de cuota", "Seña salón infantil"). Para
  `INGRESO_MANUAL` / `EGRESO_MANUAL` cumple además el rol de `descripcion` libre que pide el Flujo 7 (paso 31): no se
  declara un campo `descripcion` separado para no duplicar la misma información.
- `participante` puede ser null en movimientos sin persona asociada (Ej.: `INGRESO_MANUAL` / `EGRESO_MANUAL` o `AJUSTE`
  sin contraparte).
- **Datos de la anulación (Flujo 7):** la fila de "Anulaciones del día" se arma desde el `MovimientoCaja` de tipo
  `ANULACION`: la *hora* es su `creadoEn`, el *usuario que anuló* su `creadoPorId`, el *motivo* su `motivoAnulacion` y el
  *origen anulado* se abre vía `movimientoOriginalAnuladoId`. Se almacena `motivoAnulacion` en el propio movimiento (y no
  solo en una `CancelacionOperacion`) porque no toda anulación de caja proviene de una `CancelacionOperacion` de negocio
  (Ej.: anulación directa de un `INGRESO_MANUAL`), y el Flujo 7 exige el motivo en la grilla sin navegar a otra entidad.
- `area` (Value Object `AreaComplejo`) habilita el agrupamiento de ingresos por área que exigen los Flujos 7 y 8, sin 
  depender del texto libre de `concepto`.
- **Equivalencia de nomenclatura (Flujo 8):** el "nuevo concepto" `OrigenMovimientoCaja` (`origenTipo` + `origenId`) del
  Flujo 8 corresponde exactamente a `MovimientoCaja.origenOperacion` (`OrigenOperacion`: `tipoOrigen` + `idOrigen`). Es lo
  que usa el informe de ingresos para abrir el detalle real de la operación que generó el movimiento (Flujo 8, paso 26).
- `movimientoOriginalAnuladoId` enlaza un movimiento de tipo `ANULACION` con el movimiento original anulado, requisito
  explícito del Flujo 7 (`MovimientoOriginalAnulado`) y del Flujo 11. Permite abrir desde la anulación el movimiento de
  origen y conservar trazabilidad completa.
- `participante` permite mostrar directamente "Cliente / Persona" en la grilla de caja (Flujo 7) sin navegar al pago.
- **`área` en pagos de cuota multi-actividad:** el Flujo 3 exige un único `MovimientoCaja` por el total recibido (paso 58),
  pero un mismo pago puede saldar cuotas de actividades de distintas áreas (Ej.: Escuela de fútbol + Taekwondo). Como `área`
  es un único valor, ese `MovimientoCaja` no puede atribuirse fielmente a una sola área. Regla adoptada: el `MovimientoCaja`
  del pago lleva `area = OTRO` (ingreso por cuotas mixto) y la apertura por área de los ingresos de cuotas, cuando se
  requiera en los Flujos 7 y 8, se reconstruye desde `AplicacionPago` → `Cuota` → `Actividad`, que sí conserva la actividad
  exacta de cada parte aplicada. Si el pago salda cuotas de una sola actividad, se usa el `AreaComplejo` correspondiente.
- **`área` en pagos de Reserva/Evento:** cuando él `MovimientoCaja` se origina por un `Pago` aplicado a una `Reserva`,
  `area` se deriva del `TipoReserva`: `EDUCACION_FISICA_ESCOLAR` → `EDUCACION_FISICA`; el resto de los tipos
  (`ALQUILER_FUTBOL_5`, `RESERVA_DEPORTIVA_COMUN`, `USO_INSTITUCIONAL`, `OTRO`) → `ALQUILER_CANCHA` (Flujo 5). **No se
  fija `ALQUILER_CANCHA` para toda reserva**, porque eso clasificaría los ingresos de educación física escolar bajo
  "Alquiler de cancha" y rompería la apertura por área "Educación física" que exige el informe de ingresos por área
  (Flujo 8, sub-flujo 8.2). Sí se aplica a un `Evento`, `area` según el tipo de evento (`CUMPLEANOS_DEPORTIVO`,
  `SALON_INFANTIL`, `EVENTO_PARTICULAR` — Flujo 6). `MovimientoCajaHandler` deriva `area` del `ReferenciaCobrable` destino
  del pago. La `Venta` usa `area = CONFITERIA_CAFETERIA` directamente (Flujo 4).
- Aplicar saldo a favor a una cuota NO genera `MovimientoCaja`. Sin dinero nuevo = sin movimiento de caja.
- Registro append-only. Los movimientos anulados se marcan con `estado = ANULADO`, nunca se eliminan.
---

## OcupacionEspacio (Extiende de AuditoriaEntidad)
Bloqueo real de un espacio físico durante un rango horario concreto.

**Atributos:**

    - private Long espacioFisicoId;
    - private HorarioConcreto rangoHorario;
    - private TipoOcupacion tipoOcupacion;
    - private EstadoOcupacion estadoOcupacion;
    - private OrigenOperacion origenOperacion;
    - private String observacion;                
    - private String motivoCancelacion;          ← null salvo que estadoOcupacion = CANCELADA

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Aggregate central de disponibilidad. La agenda consulta `OcupacionEspacio`, no cada entidad por separado.
- `estadoOcupacion`: `ACTIVA`, `CANCELADA`, `REALIZADA`, `ANULADA`. Solo las `ACTIVA` bloquean el horario. `CANCELADA`
  la libera conservando historial (Flujo 5/6); `REALIZADA` marca que la ocupación ya ocurrió (Flujo 5/9).
- **Equivalencia de nomenclatura:** el Flujo 9 nombra el estado final como `FINALIZADA`; aquí se usa `REALIZADA` (mismo 
  significado y alineado con `Reserva`/`Evento`). Toda referencia a `FINALIZADA` del Flujo 9 apunta a `REALIZADA`.
- `creadoPorId` heredado de `AuditoriaEntidad` actúa como "usuario que registró la ocupación".
- **Actividad asociada (pantalla "Nueva ocupación", Flujo 9):** cuando `tipoOcupacion = ACTIVIDAD` (o `EDUCACION_FISICA`), 
  la actividad asociada NO se modela como campo propio: queda en `origenOperacion` (`tipoOrigen = ACTIVIDAD`, `idOrigen = actividadId`), 
  manteniendo bajo acoplamiento entre aggregates. Para `RESERVA` / `EVENTO`, `origenOperacion` apunta a la reserva o evento 
  que originó el bloqueo.
- **Cancelación manual (Flujo 9, sub-flujo cancelar):** la cancelación de una ocupación manual/mantenimiento/actividad fija
  no mueve dinero, por lo que NO genera `CancelacionOperacion` (esa entidad modela cancelaciones con tratamiento de dinero
  de reserva/evento/venta/pago). El `motivo` exigido por el flujo (paso 4) se guarda en `motivoCancelacion` y el cambio se
  audita vía `AuditoriaOperacion`. Para cancelaciones de la ocupación derivadas de cancelar una `Reserva`/`Evento`, el
  motivo vive en la `CancelacionOperacion` de la operación de negocio.
- La validación de disponibilidad ocurre ANTES de crear la `OcupacionEspacio` (en el servicio de aplicación via
  `ValidacionDisponibilidadDeEspacio`), no como reacción al evento `OcupacionCreada`.
---

## Reserva (Extiende de OperacionConEspacioFisico)
Alquiler puntual de un espacio físico.

**Atributos:**

    - private TipoReserva tipoReserva;
    - private EstadoReserva estadoReserva;

**Relaciones:**
- Ninguna directa. Hereda `participante` y `ocupacionId` de `OperacionConEspacioFisico`.

**Datos Importantes:**
- `importeTotal` e `importePagado` heredados de `OperacionCobrableBase`. No se redeclaran.
- **No se declara un campo `seña`.** El monto de la seña y de los pagos posteriores se registra vía `Pago` + `AplicacionPago` 
  (`ReferenciaCobrable.paraReserva(id)`) y se refleja en `importePagado`. El monto de seña concreto es reconstruible desde 
  el primer `Pago` asociado a la reserva (ver `PagoRepositorio.buscarPorCobrable`). Mantener un campo `seña` separado 
  duplicaría la fuente de verdad del cobro y podría desincronizarse de `importePagado`.
- `estadoReserva` representa el estado del proceso de reserva (PENDIENTE, SEÑADA, PAGADA, CANCELADA, REALIZADA), distinto al
  `estado` heredado de `OperacionBase` que representa el ciclo de vida de la operación (ACTIVA, ANULADA). Se deriva del saldo:
  sin pago -> `PENDIENTE`; con pago y saldo > 0 -> `SEÑADA`; saldo = 0 -> `PAGADA`. La derivación ocurre dentro de
  `registrarAplicacionPago`, en la misma operación que actualiza `importePagado`. La anulación de un pago de reserva
  entra por `revertirAplicacionPago(...)`, que resta `importePagado` y re-deriva `estadoReserva`: si el saldo vuelve a 
  ser > 0, la reserva NO puede quedar `PAGADA`.
- El saldo a favor del cliente NO se aplica a reservas en V1.
- Cancelación: ver `CancelacionOperacion` (motivo, usuario, fecha, `tratamientoDinero`) y la transición de la
  `OcupacionEspacio` asociada a `CANCELADA`. Realización: `estadoReserva = REALIZADA` y `OcupacionEspacio = REALIZADA`; el
  usuario/fecha del cambio quedan en `AuditoriaOperacion` (el Flujo 5 exige auditar el cambio a `REALIZADA`).
---

## Evento (Extiende de OperacionConEspacioFisico)
Evento o cumpleaños con datos adicionales a una reserva simple.

**Atributos:**

    - private TipoEvento tipoEvento;
    - private String nombreCumpleanero;                  ← null si el evento no es un cumpleaños
    - private Integer edadCumpleanero;                    ← null si no corresponde
    - private int cantidadInvitados;
    - private List<ServicioEvento> servicios;
    - private boolean advertenciaEdadAceptada;           ← true si se acepto la advertencia de edad orientativa
    - private String observacionAdvertenciaEdad;         ← queda registrada si el admin continua pese a la advertencia
    - private EstadoEvento estadoEvento;

**Relaciones:**
- Contiene `List< ServicioEvento >` como parte del Aggregate `Evento`.

**Datos Importantes:**
- Hereda `participante` y `ocupacionId` de `OperacionConEspacioFisico`.
- `cantidadInvitados` no puede ser negativo.
- `nombreCumpleanero` / `edadCumpleanero` son requeridos por el Flujo 6 para cumpleaños y opcionales para
  `EVENTO_PARTICULAR_INFANTIL`.
- **Validación de edad:** para `CUMPLEANOS_SALON_INFANTIL` la edad 3-7 es BLOQUEANTE (se valida en el factory
  method / servicio). Para `EVENTO_PARTICULAR_INFANTIL` en salon infantil con edad fuera de rango, la advertencia NO es
  bloqueante: se exige `advertenciaEdadAceptada = true` y se guarda `observacionAdvertenciaEdad`.
- `importeTotal` e `importePagado` heredados de `OperacionCobrableBase`. No se redeclaran. El Flujo 6 captura él
  `importeTotal` manualmente y valída que sea mayor a cero; cuando se cargan `servicios`, su suma de `subtotal` es un 
  valor de referencia para sugerir el importe, pero el `importeTotal` cobrable es el confirmado por el administrador (la 
  lista `servicios` no recalcula automáticamente el total en V1).
- **No se declara un campo `seña` ni `saldoPendiente`** (igual criterio que `Reserva`). La seña, los pagos posteriores y
  el pago final se registran vía `Pago` + `AplicacionPago` (`ReferenciaCobrable.paraEvento(id)`) y se reflejan en
  `importePagado`. El saldo pendiente es `getSaldoPendiente()` del contrato `Cobrable`. Esto cubre el concepto `PagoEvento`
  del Flujo 6 (un evento puede recibir varios pagos: seña, parcial y final) sin una entidad `PagoEvento` dedicada ni
  duplicar la fuente de verdad del cobro.
- **Derivación de `estadoEvento`:** se deriva del saldo dentro de `registrarAplicacionPago`, en la misma operación que 
  actualiza `importePagado`: sin pago → `PENDIENTE`; con pago y saldo > 0 → `SEÑADO`; saldo = 0 → `PAGADO`. La anulación 
  de un pago de evento entra por `revertirAplicacionPago(...)`, que resta `importePagado` y re-deriva `estadoEvento` (un 
  saldo > 0 impide `PAGADO`). `CANCELADO` y `REALIZADO` se asignan desde sus sub-flujos. `estadoEvento` (estado del proceso 
  del evento) es independiente del `estado` heredado de `OperacionBase` (`ACTIVA`/`ANULADA`), igual que en `Reserva`.
- **Responsable / teléfono del evento:** cuando el evento es de persona eventual, el responsable y su teléfono se almacenan 
  en `participante.eventual` (`DatosPersonaEventual.nombre` + `contacto.telefono`). Cuando el evento es de un cliente 
  registrado, se toman del responsable principal del cliente (`ResponsableDelClienteRepositorio.buscarResponsablePrincipal`). 
  No se duplican como campos propios de `Evento`; los atributos `responsable` / `telefonoResponsable` del "Impacto en entidades" 
  del Flujo 6 quedan cubiertos por esta vía.
- **Cancelación (sub-flujo Cancelar evento):** se registra una `CancelacionOperacion` (`origenOperacion` → `EVENTO`,
  `motivo`, `usuarioId`, fecha vía `creadoEn`, `tratamientoDinero`), `estadoEvento = CANCELADO` y la `OcupacionEspacio`
  asociada pasa a `CANCELADA` (libera el horario). Los pagos no se borran; cualquier devolución se registra por el flujo
  de devolución/anulación. El concepto `MotivoCancelacionEvento` del Flujo 6 queda cubierto por `CancelacionOperacion.motivo`.
- **Realización (sub-flujo Marcar como realizado):** `estadoEvento = REALIZADO` y `OcupacionEspacio = REALIZADA` (queda como
  historial). El usuario/fecha del cambio se auditan vía `AuditoriaOperacion` (cambio a `REALIZADO`). Un evento `REALIZADO` 
  con saldo pendiente sigue figurando con saldo (no implica `PAGADO`).
- **Reprogramación (sub-flujo Reprogramar evento):** se trata como cancelación del evento original + alta de uno nuevo,
  registrando `ReprogramacionOcupacion` y, si había pagos, `tratamientoDinero = REPROGRADO` en la `CancelacionOperacion`.
---

## ServicioEvento (Entidad interna del Aggregate Evento)
Servicio adicional incluido en un evento (decoración, catering, animación, etc.).

**Atributos:**

    - private Long eventoId;              
    - private String descripcion;
    - private int cantidad;
    - private Dinero precioUnitario;
    - private Dinero subtotal;             ← desnormalización documentada: cantidad * precioUnitario

**Relaciones:**
- Pertenece al Aggregate `Evento`. Ciclo de vida controlado por `Evento`.

**Datos Importantes:**
- `subtotal` es desnormalización consciente para rendimiento. Se calcula y persiste al crear o modificar el servicio.
- `cantidad` debe ser mayor a cero.
- `precioUnitario` no puede ser negativo.
---

## ReprogramacionOcupacion (Extiende de AuditoriaEntidad)
Cambio histórico de fecha u horario de una reserva o evento.

**Atributos:**

    - private OrigenOperacion origenOperacion;
    - private Long ocupacionAnteriorId;
    - private Long ocupacionNuevaId;
    - private String motivo;
    - private Long usuarioId;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Permite conservar historial de cambios de horario.
- No conviene sobrescribir simplemente la ocupación anterior.
- La ocupación anterior queda con `estadoOcupacion = CANCELADA` o `ANULADA` según la regla de negocio definida.
---

## CancelacionOperacion (Extiende de AuditoriaEntidad)
Registro histórico de la cancelación de una reserva, evento, venta o pago.

**Atributos:**

    - private OrigenOperacion origenOperacion;
    - private String motivo;
    - private TratamientoDineroCancelacion tratamientoDinero;
    - private Dinero montoDevuelto;
    - private Dinero montoASaldoFavor;
    - private Long usuarioId;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Invariante: `montoDevuelto + montoASaldoFavor` ≤ importe original de la operación.
- `tratamientoDinero`: `RETENIDO`, `DEVOLUCION_PARCIAL`, `DEVOLUCION_TOTAL`, `REPROGRAMADO`, `A_REVISAR`, `A_SALDO_FAVOR`.
- `montoDevuelto` y `montoASaldoFavor` son `Dinero.CERO` cuando `tratamientoDinero` ∈ {`RETENIDO`, `REPROGRAMADO`, `A_REVISAR`}.
- `A_SALDO_FAVOR` no es admisible para cancelaciones de `Reserva` en V1. El servicio valida qué tratamientos son válidos 
  según el tipo de operación cancelada.
- `REPROGRAMADO` indica que el dinero se traslada a una nueva reserva; el enlace con la nueva operación se resuelve en el
  flujo de reprogramación (ver `ReprogramacionOcupacion`).
---

## CategoriaProducto (Extiende de AuditoriaEntidad)
Clasificación de productos de confitería o tienda.

**Atributos:**

    - private String nombre;
    - private String descripcion;
    - private EstadoRegistro estado;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Facilita informes y filtros.
- No borrar si existen productos asociados.
---

## Producto (Extiende de AuditoriaEntidad)
Artículo vendible con precio y stock.

**Atributos:**

    - private String codigo;
    - private String nombre;
    - private Long categoriaId;
    - private Dinero precioVenta;
    - private int stockActual;
    - private int stockMinimo;
    - private boolean controlaStock;        ← si es false, la venta no valida ni descuenta stock
    - private EstadoProducto estado;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- `stockActual` se inicializa en 0. No puede ser negativo. Validar antes de registrar salidas.
- Los cambios de stock no se hacen directamente sobre `stockActual`. Siempre se registra un `MovimientoStock` y el servicio
  actualiza `stockActual` de forma atómica.
- `codigo` es **opcional** y, si se informa, debe ser único (restricción de base de datos + validación en servicio).
  El Flujo 10 (pantalla "Nuevo producto", paso 5) NO captura `codigo`: por eso NO puede ser obligatorio. Se conserva como
  dato superador (preparado para código de barras / SKU); si no se informa, se autogenera o queda null.
- **Advertencia de nombre duplicado:** al crear un producto, el sistema verifica si ya existe otro producto `ACTIVO` con 
  el mismo `nombre`; de existir, muestra "Ya existe un producto con ese nombre". Es una advertencia **NO bloqueante** (el 
  administrador puede cancelar o continuar). Validar en servicio vía `ProductoRepositorio.buscarActivosPorNombre(nombre)`. 
  A diferencia de `EspacioFisico`, el negocio NO exige aquí unicidad estricta de nombre, solo el aviso.
- `controlaStock`: si es `true` (recomendado en V1 para productos físicos), la venta valida y descuenta stock; si es
  `false` (Ej.: cafe servido, agua caliente), la venta se registra sin tocar stock. Requisito explicito del Flujo 4.
- No borrar productos vendidos. Marcar como `INACTIVO` o `SIN_STOCK`.
---

## Venta (Extiende de OperacionCobrableBase)
Operación de venta de productos.

**Atributos:**

    - private ParticipanteOperacion participante;
    - private List<DetalleVenta> detalles;
    - private MetodoPago metodoPago;                       

**Relaciones:**
- Contiene `List< DetalleVenta >` como parte del Aggregate `Venta`.

**Datos Importantes:**
- `importeTotal` heredado de `OperacionCobrableBase`. No se redeclara.
- Si se anula: revertir stock con `MovimientoStock` compensatorio de tipo `DEVOLUCION`.
- `detalles` no puede ser vacío al momento de crear la venta.
- **Pago de la venta:** la venta es la unica operación cobrable que se salda integramente al crearse. No pasa
  por la entidad `Pago`: al confirmarse, queda con `estadoCobro = SALDADO` y publica el evento `VentaRegistrada`, que
  genera él `MovimientoCaja` (INGRESO, área `CONFITERIA_CAFETERIA`) y los `MovimientoStock` de salida. `metodoPago` queda
  en la propia venta y se replica en él `MovimientoCaja`.
---

## DetalleVenta (Extiende de AuditoriaEntidad)
Línea individual de una venta.

**Atributos:**

    - private Long ventaId;
    - private Long productoId;              
    - private String nombreProducto;
    - private int cantidad;
    - private Dinero precioUnitario;        
    - private Dinero subtotal;              

**Relaciones:**
- Pertenece al Aggregate `Venta`. Ciclo de vida controlado por `Venta`.

**Datos Importantes:**
- `nombreProducto` como snapshot evita que el nombre histórico cambie si el producto se renombra.
- `precioUnitario` congelado al momento de la venta.
- `subtotal` es desnormalización consciente para rendimiento. Nunca actualizar individualmente.
- `cantidad` debe ser mayor a cero.
---

## MovimientoStock (Extiende de AuditoriaEntidad)
Entrada, salida o ajuste histórico de stock.

**Atributos:**

    - private Long productoId;
    - private TipoMovimientoStock tipoMovimientoStock;
    - private int cantidad;
    - private int stockAnterior;
    - private int stockPosterior;
    - private OrigenOperacion origenOperacion;
    - private String observacion;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Cada cambio de stock debe dejar historial. No alcanza con modificar `stockActual` en `Producto`.
- Registro append-only.
- **Motivo, fecha y usuario del ajuste:** el `motivo` ingresado se guarda en `observacion`; la fecha/hora en `creadoEn` 
  y el usuario en `creadoPorId` (heredados de `AuditoriaEntidad`); la `cantidad` en `cantidad`. No se declara un campo 
  `motivo` separado para no duplicar `observacion`.
- `stockPosterior = stockAnterior + cantidad` (ENTRADA / AJUSTE_POSITIVO / DEVOLUCION) o `stockAnterior - cantidad`
  (SALIDA_VENTA / AJUSTE_NEGATIVO / **PERDIDA**).
---

## ConfiguracionSistema (Extiende de AuditoriaEntidad)
Parámetros generales del sistema.

**Atributos:**

    - private String nombreNegocio;
    - private ZoneId zonaHoraria;
    - private Contacto datosContacto;
    - private PreferenciasSistema preferencias;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Evita valores fijos escritos directamente en el código ("magic values").
- Permite cambiar datos generales del negocio sin recompilar.
- Singleton lógico: solo debe existir un registro activo. Consultar vía `ConfiguracionSistemaRepositorio.obtener()`.
---

## ConfiguracionCuotas (Extiende de AuditoriaEntidad)
Reglas configurables para cuotas.

**Atributos:**

    - private int diaVencimiento;
    - private boolean generarSoloActivos;
    - private boolean permitirSaldoFavor;
    - private boolean aplicarSaldoFavorAutomatico; 
    - private boolean primerDiaHabilDelMes;         
    - private RecargoPorMora recargoMora;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Usa `RecargoPorMora` en lugar de `Dinero` para soportar tanto montos fijos como porcentajes.
- Permite cambiar reglas del negocio sin tocar código.
- `diaVencimiento`: **1-28** (no 1-31). El límite en 28 evita fechas inválidas en meses cortos (febrero). Esta restricción
  aplica tanto al valor por defecto del sistema como al `diaVencimiento` de cada `Inscripcion`.
- `generarSoloActivos`: si true, evita generar cuotas a inscripciones dadas de baja.
- `permitirSaldoFavor`: define si el sistema permite usar saldo a favor para cancelar cuotas.
- `aplicarSaldoFavorAutomatico`: define si la opción de aplicación automática de saldo a favor aparece activada por
  defecto en la pantalla de generación mensual. El administrador puede modificarla por ejecución. En V1 el valor inicial
  es `true`.
- `primerDiaHabilDelMes`: si true, el sistema no permite generar cuotas hasta el primer día hábil (lunes a viernes) del
  mes. Si el día 1 cae sábado o domingo, se habilita el lunes siguiente. En V1 no se consideran feriados. En versiones
  futuras podrá incorporarse una tabla de feriados o política configurable por actividad.
- Singleton lógico: solo debe existir un registro activo.
---

## SecuenciaComprobante (Extiende de AuditoriaEntidad)
Controla la numeración correlativa de comprobantes para cada tipo.

**Atributos:**

    - private TipoComprobante tipoComprobante;
    - private String prefijo;
    - private Long proximoNumero;
    - private boolean activo;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Centraliza la numeración para evitar duplicados.
- `proximoNumero` se inicializa en 1.
- `prefijo` permite generar números como `REC-000001` o `TCK-000001`.
- **La obtención y actualización del número debe ser atómica**: usar `SELECT FOR UPDATE` o mecanismo equivalente (Ej.:
  `SecuenciaComprobanteRepositorio.obtenerYIncrementar(tipo)`) para garantizar unicidad bajo concurrencia.
- Existe una instancia por `TipoComprobante`.
---

## ConfiguracionBackup (Extiende de AuditoriaEntidad)
Parámetros de backup: destino, frecuencia y retención.

**Atributos:**

    - private DestinoBackup destino;
    - private FrecuenciaBackup frecuencia;
    - private int cantidadRetener;
    - private boolean activo;

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- El backup pertenece a infraestructura, pero sus parámetros deben ser administrables desde el dominio.
- Permite cambiar de backup local a otro destino sin modificar el código de infraestructura.
- `cantidadRetener` define cuántos backups antiguos se conservan antes de eliminar los más viejos.
- Si `activo = false`, no debe ejecutarse el backup programado.
---

## BackupSistema (Extiende de AuditoriaEntidad)
Registro de un backup generado.

**Atributos:**

    - private TipoBackup tipoBackup;
    - private UbicacionBackup ubicacion;
    - private String checksum;
    - private EstadoBackup estadoBackup;
    - private Long usuarioId;
    - private String observacion;                
    
    (creadoEn heredado actúa como fecha del backup).

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- No alcanza con crear un archivo; hay que registrar si fue exitoso y dónde quedó.
- `checksum` sirve para validar integridad antes de restaurar.
- `estadoBackup`: `EXITOSO`, `FALLIDO`, `INCOMPLETO`.
- `usuarioId` es null para backups automáticos (batch). Para backups manuales, registra quién lo inició.
- `observacion`: nota libre del resultado (Ej.: causa de fallo, tamaño, destino externo confirmado).
- Se consulta vía `BackupSistemaRepositorio` para la pantalla "Historial de backups" y para el aviso de
  backup automático fallido en el próximo inicio.
---

## RestauracionBackup (Extiende de AuditoriaEntidad)
Registro histórico de una restauración del sistema.

**Atributos:**

    - private Long backupSistemaId;
    - private Long usuarioId;
    - private EstadoRestauracionBackup estadoRestauracion;
    - private ResultadoRestauracion resultado;
    - private String observacion;

    (creadoEn heredado actúa como fecha de restauración).

**Relaciones:**
- Ninguna directa.

**Datos Importantes:**
- Restaurar datos es una operación crítica. Debe quedar registrado quién restauró, cuándo y con qué resultado.
- Antes de restaurar, validar el `checksum` del backup mediante `ProveedorBackup.validar(ubicacion, checksum)`.
- Si falla una restauración, también debe quedar registrada con `estadoRestauracion = FALLIDA`.
- Registro append-only.
---
#
#
#
#
#
# 4. INTERFACES DE DOMINIO
#

## Cobrable (Contrato de entidad)
Contrato de lectura para cualquier entidad que pueda tener saldo pendiente. Define solo métodos de consulta.

**Métodos:**

    - Dinero getImporteTotal();
    - Dinero getImportePagado();
    - Dinero getSaldoPendiente();
    - boolean estaSaldado();

**Datos Importantes:**
- Implementado por `Cuota`, `Reserva`, `Evento`, `Venta`.
- Solo lectura (ISP). La mutación del estado de cobro se gestiona a través de `ReceptorDePago`. `estaSaldado()` devuelve 
  true cuando `getSaldoPendiente()` es cero.
- `getSaldoPendiente()` = `getImporteTotal() - getImportePagado()`.
---

## ReceptorDePago (Contrato de mutación)
Contrato de escritura para entidades que pueden recibir la aplicación de un pago.

**Métodos:**

    - void registrarAplicacionPago(AplicacionPago aplicacion);

**Datos Importantes:**
- Implementado por `OperacionCobrableBase` (y por herencia: `Cuota`, `Reserva`, `Evento`, `Venta`).
- Separado de `Cobrable` para cumplir ISP: quien solo necesite consultar saldos no depende de la capacidad de mutación.
- `registrarAplicacionPago` actualiza `importePagado` y recalcula `estadoCobro` internamente en la entidad. En `Cuota`,
  el override de este método deriva además `estadoCuota` y `estadoVencimiento` en la misma operación, de modo que las tres
  proyecciones de estado quedan consistentes con el nuevo saldo pendiente sin escrituras separadas.
- Este contrato cubre solo el **pago real**. La aplicación de **saldo a favor** sobre una `Cuota` NO se modela
  como `AplicacionPago` y por eso no entra por esta interfaz: usa el método propio `Cuota.registrarAplicacionSaldoFavor(Dinero)`
  (ver `Cuota`), que mantiene la misma garantía de consistencia de estados.
---

## ValidacionDisponibilidadDeEspacio (Domain Service Interface)
Determina si un espacio físico está disponible en un rango horario dado.

**Métodos:**

    - boolean validarDisponible(EspacioFisico espacio, HorarioConcreto horario);
    - List<OcupacionEspacio> buscarConflictos(EspacioFisico espacio, HorarioConcreto horario);

**Datos Importantes:**
- Es un Domain Service (tiene implementación con lógica de negocio), definido como interfaz para permitir testeo sin
  infraestructura.
- Toma `Long espacioFisicoId` en lugar del objeto completo para reducir acoplamiento entre Aggregates.
- Se invoca ANTES de crear una `OcupacionEspacio`, como precondición del caso de uso, no como reacción al evento.
- Se puede extender para soportar `HorarioRecurrente` con una sobrecarga del método:
  `List< HorarioConcreto > detectarConflictosRecurrentes(Long espacioFisicoId, HorarioRecurrente patron, LocalDate desde, LocalDate hasta)`.
---

## PoliticaDeGeneracionCuota (Strategy - Dominio)
Estrategia que define cómo se generan las cuotas de los clientes para un período determinado.

**Métodos:**

    - List<Cuota> generarParaPeriodo(PeriodoMensual periodo);
    - List<Cuota> previsualizar(PeriodoMensual periodo);

**Dependencias declaradas (inyectadas en la implementación):**
- `InscripcionRepositorio`
- `CuotaRepositorio`
- `PrecioMensualActividadRepositorio`
- `ConfiguracionCuotasRepositorio`

**Datos Importantes:**
- Las dependencias son Ports de repositorio (DIP). La implementación concreta se inyecta desde infraestructura.
- **Resolución del importe de cada cuota:** el importe se determina con esta prioridad, para conciliar el "precio del mes 
  por actividad" con él `precioMensualPactado` por inscripción:
    1) `Inscripcion.precioMensualPactado` si no es null; 
    2) `PrecioMensualActividad.buscarVigente(actividadId, periodo)`;
    3) `Actividad.precioPorDefecto`. Como consecuencia, el "total bruto" estimado por actividad en la pantalla del Flujo 2
       es exacto solo si ninguna inscripción tiene precio pactado propio; cuando lo hay, se calcula sumando el importe real
       resuelto por inscripción, no `precioDelMes × inscriptos`.
---

## PoliticaDeAplicacionSaldoFavor (Strategy - Dominio)
Estrategia que define cómo se aplica el saldo a favor de un cliente. El saldo a favor solo puede aplicarse a cuotas.

**Métodos:**

    - void aplicarSaldo(Long clienteId, Long cuotaId, Dinero monto);
    - void validarAplicacion(Long clienteId, Long cuotaId, Dinero monto);

**Dependencias declaradas (inyectadas en la implementación):**
- `SaldoAFavorClienteRepositorio`
- `CuotaRepositorio`

**Datos Importantes:**
- Toma ID's en lugar de objetos directos para reducir el acoplamiento del strategy al Aggregate.
- Las dependencias son Ports de repositorio (DIP).
- **Criterio de orden de aplicación:** cuando un cliente tiene varias cuotas nuevas en el mismo período y su saldo a favor 
  supera el importe de una de ellas, el saldo se aplica primero a las cuotas del período actual y, dentro del mismo período, 
  en el orden definido por actividad y luego por inscripción. No se aplica automáticamente a cuotas de períodos anteriores. 
  Por cada cuota el monto aplicado es el menor entre el saldo disponible del cliente y el saldo pendiente de la cuota; el 
  sobrante queda disponible para la siguiente cuota del orden. La orquestación del recorrido ordenado es responsabilidad 
  del servicio de aplicación, que invoca `aplicarSaldo(...)` por cada cuota siguiendo este criterio.
---

## PoliticaDeDistribucionPago (Strategy - Dominio)
Estrategia que define cómo se reparte un pago entre una o varias deudas pendientes.

**Métodos:**

    - void distribuir(Long pagoId, List<ReferenciaCobrable> destinos);
    - void validarDistribucion(Long pagoId, List<ReferenciaCobrable> destinos);

**Dependencias declaradas (inyectadas en la implementación):**
- `PagoRepositorio`
- `CuotaRepositorio`

**Datos Importantes:**
- Las dependencias son Ports de repositorio (DIP).
- **Criterio de orden de distribución:** cuando el monto recibido es menor al total seleccionado, el pago se reparte entre 
  las cuotas seleccionadas en este orden estricto:
    1. Primero las cuotas con `estadoVencimiento = VENCIDA`.
    2. Luego las cuotas pendientes más antiguas (por `periodo` ascendente).
    3. Luego las cuotas más recientes.
       Por cada cuota, el monto aplicado es el menor entre el remanente del pago y el saldo pendiente de la cuota; el sobrante
       pasa a la siguiente cuota del orden. La distribución resultante debe poder previsualizarse antes de confirmar. Como 
       mejora futura, el sistema podrá permitir que el administrador defina manualmente cuánto aplicar a cada cuota. 
       El recorrido ordenado lo orquesta el servicio de aplicación, que registra una `AplicacionPago` por cada cuota alcanzada.
---
#
#
#
#
#
# 5. PORTS DE INFRAESTRUCTURA (DIP - Interfaces definidas por el dominio)
#

## PasswordHasher (Port de seguridad)
Puerto de seguridad responsable de hashear y verificar contraseñas sin acoplar el dominio a una librería concreta.

**Métodos:**

    - String hash(String raw);
    - boolean matches(String raw, String hash);
---

## AuditoriaPort (Port de salida)
Puerto de salida que permite registrar eventos de auditoría desde los servicios de aplicación sin que el dominio conozca
dónde se guardan.

**Métodos:**

    - void registrar(AuditoriaOperacion evento);
---

## AuditoriaConsultaPort (Port de lectura)
Puerto de lectura para la pantalla "Consulta de auditoría" del Flujo 12. Separado de `AuditoriaPort` (escritura) para
respetar ISP/CQRS: quien solo registra no depende de la capacidad de consulta y viceversa.

**Métodos:**

    - List<AuditoriaOperacion> buscar(FiltroAuditoria filtro);

**Datos Importantes:**
- `FiltroAuditoria` agrupa los filtros de la pantalla: `fechaDesde`, `fechaHasta`, `usuarioId` (null = todos)
  y `tipoOperacion` (null = todas). Cualquier filtro null se ignora.
- Sin este puerto la auditoría sería append-only sin forma de cumplir el requisito de consulta filtrada del Flujo 12.
---

## ProveedorBackup (Port de infraestructura)
Puerto de infraestructura encargado de crear, validar y restaurar respaldos del sistema.

**Métodos:**

    - ResultadoBackup crearBackup(ConfiguracionBackup config);
    - boolean validar(UbicacionBackup ubicacion, String checksum);
    - ResultadoRestauracion restaurar(ArchivoBackup archivo);

**Datos Importantes:**
- `crearBackup` recibe la `ConfiguracionBackup` vigente (destino, frecuencia) y **devuelve** un `ResultadoBackup`
  (`UbicacionBackup` + `checksum` + `EstadoBackup` + observación) para que el servicio de aplicación pueda persistir el
  registro `BackupSistema` con esos datos.
- `restaurar` devuelve un `ResultadoRestauracion` para poder registrar `RestauracionBackup` con su resultado real
  (`COMPLETA`, `PARCIAL`, `FALLIDA`).
- `validar` se usa antes de restaurar para verificar el `checksum` del backup elegido.
---

## ConfiguracionBackupRepositorio (Port de repositorio)

**Métodos:**

    - Optional<ConfiguracionBackup> obtener();
    - void guardar(ConfiguracionBackup config);

**Datos Importantes:**
- Singleton lógico (mismo patrón que `ConfiguracionSistemaRepositorio`): existe una sola configuración de backup activa.
- El sub-flujo de backup automático (Flujo 13) la consulta para conocer destino, frecuencia y si `activo = true`.
---

## BackupSistemaRepositorio (Port de repositorio)

**Métodos:**

    - Optional<BackupSistema> buscarPorId(Long id);
    - Optional<BackupSistema> buscarUltimo();
    - List<BackupSistema> buscarRecientes(int cantidad);
    - List<BackupSistema> buscarPorRango(LocalDate desde, LocalDate hasta);
    - void guardar(BackupSistema backup);

**Datos Importantes:**
- Sirve la pantalla "Historial de backups" del Flujo 13 (lista de backups con fecha, usuario, tipo, estado, ubicación).
- `buscarUltimo` alimenta el bloque "Último backup realizado" de la pantalla de respaldo y el aviso de backup
  automático fallido en el próximo inicio.
---

## RestauracionBackupRepositorio (Port de repositorio)

**Métodos:**

    - Optional<RestauracionBackup> buscarPorId(Long id);
    - List<RestauracionBackup> buscarRecientes(int cantidad);
    - void guardar(RestauracionBackup restauracion);

**Datos Importantes:**
- Registro append-only de las restauraciones. Conserva quién restauró, cuándo, desde qué backup y con qué resultado.
---

## ClienteRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Cliente> buscarPorId(Long id);
    - Optional<Cliente> buscarPorDocumento(DocumentoIdentidad documento);
    - List<Cliente> buscarPorNombreOApellido(String texto);           ← búsqueda parcial por nombre o apellido
    - List<Cliente> buscarPosiblesDuplicados(NombrePersona nombre, LocalDate fechaNacimiento, Long responsableId);  ← responsableId puede ser null
    - List<Cliente> buscarActivos();
    - void guardar(Cliente cliente);

**Datos Importantes:**
- `buscarPorNombreOApellido(String texto)`: permite al administrador buscar un cliente por nombre, apellido o DNI. La 
  búsqueda por DNI ya está cubierta por `buscarPorDocumento`. La búsqueda textual por nombre o apellido se resuelve con 
  este método. La estrategia de coincidencia (exacta, `LIKE`, o fuzzy) es responsabilidad de la implementación de 
  infraestructura. Devuelve `List` porque puede haber varios clientes con nombres similares; el administrador selecciona 
  el correcto desde la UI.
- `buscarPosiblesDuplicados`: usado cuando el cliente se registra sin DNI. Busca coincidencias aproximadas por
  nombre + apellido + fecha de nacimiento, y opcionalmente filtra/prioriza por `responsableId` si ya hay un responsable
  cargado en el formulario al momento de la búsqueda. `responsableId` puede ser `null` si todavía no se cargó responsable.
  La estrategia de "coincidencia aproximada" (exacta vs. fuzzy matching) es responsabilidad de la implementación de
  infraestructura, no del contrato del puerto.
---

## ResponsableRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Responsable> buscarPorId(Long id);
    - Optional<Responsable> buscarPorDocumento(DocumentoIdentidad documento);
    - List<Responsable> buscarPorTelefono(String telefono);              
    - List<Responsable> buscarPorNombreYApellido(String nombre, String apellido);  
    - void guardar(Responsable responsable);

**Datos Importantes:**
- `buscarPorTelefono` y `buscarPorNombreYApellido` soportan la búsqueda de responsable existente del flujo de alta de
  clientes (búsqueda por DNI, teléfono, o nombre y apellido), evitando la creación de responsables duplicados.
- Devuelven `List` (no `Optional`) porque tanto el teléfono como el nombre/apellido pueden coincidir con más de un
  responsable; el administrador elige cuál corresponde desde la UI.
---

## ResponsableDelClienteRepositorio (Port de repositorio)

**Métodos:**

    - Optional<ResponsableDelCliente> buscarPorId(Long id);
    - List<ResponsableDelCliente> buscarPorCliente(Long clienteId);
    - List<ResponsableDelCliente> buscarPorResponsable(Long responsableId);
    - Optional<ResponsableDelCliente> buscarResponsablePrincipal(Long clienteId);
    - void guardar(ResponsableDelCliente vinculo);

**Datos Importantes:**
- `buscarResponsablePrincipal`: permite verificar de forma eficiente si ya existe un responsable principal activo
  para un cliente, en lugar de traer toda la lista de vínculos y filtrar en memoria. Soporta la restricción de negocio
  de unicidad del responsable principal documentada en `ResponsableDelCliente`.
---

## RolRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Rol> buscarPorId(Long id);
    - Optional<Rol> buscarPorNombre(String nombre);
    - List<Rol> buscarTodos();
    - void guardar(Rol rol);

**Datos Importantes:**
- `buscarPorNombre` permite resolver roles fijos (ADMINISTRADOR, ENCARGADO, EMPLEADO, CONSULTA) para asignación y para
  la regla del último administrador.
- No se define `PermisoRepositorio`: `Permiso` pertenece al aggregate `Rol` y no se consulta de forma independiente.
---

## UsuarioRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Usuario> buscarPorId(Long id);
    - Optional<Usuario> buscarPorUsername(String username);
    - List<Usuario> buscarActivos();
    - List<Usuario> buscarActivosPorRol(Long rolId);         
    - long contarActivosPorRol(Long rolId);                   ← validación eficiente de no desactivar el último ADMIN
    - void guardar(Usuario usuario);

**Datos Importantes:**
- `buscarActivosPorRol` / `contarActivosPorRol` soportan la regla de negocio("No se puede desactivar el último administrador
  activo"): el servicio resuelve el `rolId` de ADMINISTRADOR vía `RolRepositorio.buscarPorNombre` y verifica que quede al 
  menos un usuario activo con ese rol antes de desactivar o cambiar de rol.
---

## SesionRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Sesion> buscarPorToken(String token);
    - List<Sesion> buscarActivas(Long usuarioId);
    - void guardar(Sesion sesion);
---

## EspacioFisicoRepositorio (Port de repositorio)

**Métodos:**

    - Optional<EspacioFisico> buscarPorId(Long id);
    - List<EspacioFisico> buscarActivos();
    - void guardar(EspacioFisico espacio);
---

## ActividadRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Actividad> buscarPorId(Long id);
    - List<Actividad> buscarActivas();
    - List<Actividad> buscarQueGeneranCuota();
    - void guardar(Actividad actividad);
---

## PrecioMensualActividadRepositorio (Port de repositorio)

**Métodos:**

    - Optional<PrecioMensualActividad> buscarVigente(Long actividadId, PeriodoMensual periodo);
    - List<PrecioMensualActividad> buscarHistorial(Long actividadId);
    - void guardar(PrecioMensualActividad precio);
---

## InscripcionRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Inscripcion> buscarPorId(Long id);
    - List<Inscripcion> buscarActivasPorCliente(Long clienteId);
    - List<Inscripcion> buscarTodasActivas();                          ← para generación masiva de cuotas
    - List<Inscripcion> buscarActivasPorActividad(Long actividadId);   ← para generación por actividad
    - boolean existeInscripcionActiva(Long clienteId, Long actividadId);
    - void guardar(Inscripcion inscripcion);

**Datos Importantes:**
- `buscarTodasActivas()` es requerido por el proceso de generación mensual masiva de cuotas, donde el sistema
  debe procesar todas las inscripciones activas de todas las actividades en una única transacción.
- `buscarActivasPorActividad(Long actividadId)` es requerido para construir el resumen por actividad que se muestra en la
  pantalla de generación y previsualización.
---

## CuotaRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Cuota> buscarPorId(Long id);
    - List<Cuota> buscarPendientesPorCliente(Long clienteId);
    - List<Cuota> buscarPorPeriodo(PeriodoMensual periodo);            ← para resumen de generación
    - boolean existeParaPeriodo(Long clienteId, Long actividadId, PeriodoMensual periodo);
    - void guardar(Cuota cuota);

**Datos Importantes:**
- `buscarPorPeriodo(PeriodoMensual periodo)` permite obtener todas las cuotas generadas para un período dado, necesario
  para calcular totales del resumen final y para el sub-flujo de detección de cuotas faltantes.
---

## PagoRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Pago> buscarPorId(Long id);
    - List<Pago> buscarPorCliente(Long clienteId);
    - List<Pago> buscarPorCobrable(TipoCobrable tipo, Long cobradoId);   ← pagos aplicados a una Cuota/Reserva/Evento
    - void guardar(Pago pago);

**Datos Importantes:**
- `buscarPorCobrable(tipo, cobradoId)` devuelve todos los `Pago` con al menos una `AplicacionPago` cuyo `destino` apunta a
  esa entidad. Requerido por el sub-flujo de pago posterior de reserva y por la ficha de pagos de una cuota. La estrategia 
  de consulta (join por `AplicacionPago.destino`) es responsabilidad de la implementación de infraestructura.
---

## OcupacionEspacioRepositorio (Port de repositorio)

**Métodos:**

    - List<OcupacionEspacio> buscarActivasPorEspacio(Long espacioFisicoId, LocalDate fecha);
    - boolean existeConflicto(Long espacioFisicoId, HorarioConcreto horario);
    - void guardar(OcupacionEspacio ocupacion);
---

## ReservaRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Reserva> buscarPorId(Long id);
    - List<Reserva> buscarPorCliente(Long clienteId);
    - List<Reserva> buscarPorRango(LocalDate desde, LocalDate hasta, EstadoReserva estado);   ← estado null = todos
    - void guardar(Reserva reserva);

**Datos Importantes:**
- `buscarPorRango` da soporte al "Informe de reservas de cancha", que filtra por rango de fechas y, opcionalmente, por 
  `EstadoReserva`. La fecha de la reserva se resuelve por la `OcupacionEspacio` asociada (`HorarioConcreto.fecha`); la 
  estrategia de join es responsabilidad de infraestructura.
---

## EventoRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Evento> buscarPorId(Long id);
    - List<Evento> buscarPorCliente(Long clienteId);
    - List<Evento> buscarPorRango(LocalDate desde, LocalDate hasta, TipoEvento tipo, EstadoEvento estado);   ← tipo/estado null = todos
    - void guardar(Evento evento);

**Datos Importantes:**
- `buscarPorRango` da soporte al "Informe de eventos/cumpleaños", que filtra por rango de fechas y, opcionalmente, por 
  `TipoEvento` y `EstadoEvento`.
---

## VentaRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Venta> buscarPorId(Long id);
    - List<Venta> buscarPorCliente(Long clienteId);
    - List<Venta> buscarPorRango(LocalDate desde, LocalDate hasta);   
    - void guardar(Venta venta);

**Datos Importantes:**
- `buscarPorRango` da soporte al "Informe de ventas de confitería/cafetería". El "Informe de productos más
  vendidos" se calcula agregando los `DetalleVenta` de las ventas del período.
---

## ProductoRepositorio (Port de repositorio)

**Métodos:**

    - Optional<Producto> buscarPorId(Long id);
    - Optional<Producto> buscarPorCodigo(String codigo);
    - List<Producto> buscarActivos();
    - List<Producto> buscarActivosPorNombre(String nombre);   
    - List<Producto> buscarConStockBajo();         controlaStock = true y stockActual <= stockMinimo
    - void guardar(Producto producto);

**Datos Importantes:**
- `buscarConStockBajo` da soporte al "Informe de stock bajo": productos activos con control de stock cuyo
  `stockActual <= stockMinimo`.
---

## MovimientoStockRepositorio (Port de repositorio)

**Métodos:**

    - List<MovimientoStock> buscarPorProducto(Long productoId);
    - void guardar(MovimientoStock movimiento);
---

## CajaDiariaRepositorio (Port de repositorio)

**Métodos:**

    - Optional<CajaDiaria> buscarAbierta();
    - Optional<CajaDiaria> buscarPorFecha(LocalDate fecha);
    - void guardar(CajaDiaria cajaDiaria);
---

## MovimientoCajaRepositorio (Port de repositorio)

**Métodos:**

    - List<MovimientoCaja> buscarPorCajaDiaria(Long cajaDiariaId);
    - List<MovimientoCaja> buscarPorOrigen(TipoOrigenOperacion tipo, Long idOrigen);
    - List<MovimientoCaja> buscarPorRango(LocalDate desde, LocalDate hasta, EstadoMovimientoCaja estado);   ← estado null = todos
    - void guardar(MovimientoCaja movimiento);

**Datos Importantes:**
- `buscarPorCajaDiaria` permite obtener todos los movimientos de una jornada para el resumen de caja.
- `buscarPorOrigen` permite trazar el movimiento de caja generado por un pago concreto (Ej.: todos los movimientos
  originados por `Pago #15`). Necesario para auditoría y para la pantalla de ficha del pago.
- `buscarPorRango` da soporte a los informes de ingresos del Flujo 8 (diario, por área y por método de pago), que se
  basan en movimientos de caja del período. El agrupamiento por `AreaComplejo` y por `MetodoPago` y la exclusión de los
  `estado = ANULADO` de los totales normales se resuelven sobre el resultado de esta consulta.
- Registro append-only. Solo se guardan movimientos nuevos; los anulados se marcan con `estado = ANULADO`.
---

## SaldoAFavorClienteRepositorio (Port de repositorio)

**Métodos:**

    - Optional<SaldoAFavorCliente> buscarPorCliente(Long clienteId);
    - List<SaldoAFavorCliente> buscarConSaldoPositivo();      ← clientes con saldo a favor > 0
    - void guardar(SaldoAFavorCliente saldo);

**Datos Importantes:**
- `buscarConSaldoPositivo` da soporte al "Informe de saldos a favor por cliente": clientes cuyo `montoDisponible` es mayor 
  a `Dinero.CERO`.
---

## ConfiguracionSistemaRepositorio (Port de repositorio)

**Métodos:**

    - ConfiguracionSistema obtener();
    - void guardar(ConfiguracionSistema configuracion);
---

## ConfiguracionCuotasRepositorio (Port de repositorio)

**Métodos:**

    - ConfiguracionCuotas obtener();
    - void guardar(ConfiguracionCuotas configuracion);
---

## GeneracionCuotaRepositorio (Port de repositorio)
Corresponde al repositorio de `GeneracionCuotasMensuales` según la nomenclatura del Flujo 2.

**Métodos:**

    - Optional<GeneracionCuota> buscarPorId(Long id);
    - Optional<GeneracionCuota> buscarConfirmadaPorPeriodo(PeriodoMensual periodo);
    - Optional<GeneracionCuota> buscarEnProcesoPorPeriodo(PeriodoMensual periodo);
    - List<GeneracionCuota> buscarPorPeriodo(PeriodoMensual periodo);  ← incluye normales y complementarias
    - boolean existeConfirmadaParaPeriodo(PeriodoMensual periodo);
    - void guardar(GeneracionCuota generacion);

**Datos Importantes:**
- `buscarConfirmadaPorPeriodo` permite verificar si ya existe una generación confirmada antes de iniciar una nueva.
- `buscarEnProcesoPorPeriodo` permite detectar procesos colgados o en ejecución concurrente del mismo período.
- `existeConfirmadaParaPeriodo` es la verificación rápida usada en la pantalla principal para mostrar u ocultar el aviso
  de generación de cuotas.
---

## DetalleGeneracionCuotaRepositorio (Port de repositorio)

**Métodos:**

    - List<DetalleGeneracionCuota> buscarPorGeneracion(Long generacionId);
    - List<DetalleGeneracionCuota> buscarOmitidasPorGeneracion(Long generacionId);
    - void guardar(DetalleGeneracionCuota detalle);

**Datos Importantes:**
- `buscarOmitidasPorGeneracion` facilita la construcción del resumen de omisiones en la pantalla de resultado final, sin
  necesidad de filtrar en memoria la lista completa de detalles.
---

## MovimientoSaldoClienteRepositorio (Port de repositorio)

**Métodos:**

    - List<MovimientoSaldoCliente> buscarPorCliente(Long clienteId);
    - void guardar(MovimientoSaldoCliente movimiento);

**Datos Importantes:**
- Registro append-only. Solo se guardan movimientos nuevos; nunca se modifican existentes.
- `buscarPorCliente` permite consultar el historial completo de movimientos de saldo de un cliente.
---

## AplicacionSaldoFavorCuotaRepositorio (Port de repositorio)

**Métodos:**

    - List<AplicacionSaldoFavorCuota> buscarPorCuota(Long cuotaId);
    - List<AplicacionSaldoFavorCuota> buscarPorCliente(Long clienteId);
    - List<AplicacionSaldoFavorCuota> buscarPorRango(LocalDate desde, LocalDate hasta); 
    - void guardar(AplicacionSaldoFavorCuota aplicacion);

**Datos Importantes:**
- Registro append-only. Cada aplicación de saldo queda registrada permanentemente.
- `buscarPorCuota` permite ver el historial completo de aplicaciones de saldo sobre una cuota específica.
- `buscarPorRango` da soporte al "Informe de aplicaciones de saldo a favor", que totaliza el saldo aplicado
  en un período. La fecha de aplicación es `creadoEn` heredado.
---

## SecuenciaComprobanteRepositorio (Port de repositorio)

**Métodos:**

    - String obtenerYIncrementar(TipoComprobante tipo);    ← operación atómica (SELECT FOR UPDATE internamente)
    - void guardar(SecuenciaComprobante secuencia);
---

## ComprobanteOperacionRepositorio (Port de repositorio)

**Métodos:**

    - Optional<ComprobanteOperacion> buscarPorId(Long id);
    - Optional<ComprobanteOperacion> buscarPorOrigen(TipoOrigenOperacion tipo, Long idOrigen);
    - List<ComprobanteOperacion> buscarPorCliente(Long clienteId);
    - void guardar(ComprobanteOperacion comprobante);

**Datos Importantes:**
- `buscarPorOrigen` permite recuperar el comprobante emitido para un pago concreto (Ej.: el recibo del `Pago #15`).
  Es el método que usa la pantalla de "Ver comprobante" al finalizar el flujo de cobro.
- `buscarPorCliente` permite listar todos los comprobantes emitidos para un cliente desde su ficha.
- Un comprobante anulado (`estado = ANULADO`) no puede reactivarse; la operación de anulación solo actualiza el estado.
---
#
#
#
#
#
# 6. DOMAIN EVENTS
#

## PagoRegistrado
Publicado cuando un pago es confirmado exitosamente.

**Datos del evento:**

    - Long pagoId;
    - Long clienteId;              ← null si es persona eventual
    - Dinero totalRecibido;
    - List<ReferenciaCobrable> destinos;
    - List<DetalleMetodoPago> metodosPago;   ← necesario para que MovimientoCajaHandler construya MovimientoCaja con MetodoPago
    - Dinero excedenteSaldoFavor;            ← Dinero.CERO si no se generó saldo a favor
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `MovimientoCajaHandler` → genera `MovimientoCaja` usando `totalRecibido` y `metodosPago`.
- `AplicacionPagoHandler` → aplica monto a cada `Cobrable` destino.
- `SaldoFavorHandler` → si `excedenteSaldoFavor > Dinero.CERO`, actualiza `SaldoAFavorCliente` y registra
  `MovimientoSaldoCliente` de tipo `GENERACION_SALDO_A_FAVOR`.
- `ComprobanteHandler` → emite `ComprobanteOperacion`.
- `AuditoriaHandler` → registra `AuditoriaOperacion` con todos los datos del pago (usuario, cuotas afectadas, saldo
  a favor generado, origen).
---

## PagoAnulado
Publicado cuando un pago es anulado.

**Datos del evento:**

    - Long pagoId;
    - Long clienteId;                                   ← null si era persona eventual
    - Dinero montoOriginal;
    - List<ReferenciaCobrable> aplicacionesAfectadas;   ← incluye destinos CUOTA, RESERVA y EVENTO
    - Dinero saldoFavorGeneradoOriginal;                ← Dinero.CERO si el pago original no generó saldo a favor
    - Long usuarioAnulacionId;                          ← usuario que anuló
    - String motivo;                                    ← motivo obligatorio de la anulación
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `MovimientoCajaHandler` → genera `MovimientoCaja` de tipo `ANULACION` por `montoOriginal`, referenciando el movimiento
  original vía `movimientoOriginalAnuladoId` y guardando `motivoAnulacion`.
- `CuotaHandler` → revierte `importePagado` de las cuotas afectadas (destinos `CUOTA`) vía `Cuota.revertirAplicacionPago(...)`,
  recalculando `estadoCuota` y `estadoVencimiento` (puede volver a `VENCIDA`).
- `ReservaEventoHandler` → revierte `importePagado` de la `Reserva`/`Evento` afectado (destinos `RESERVA`/`EVENTO`) vía
  `revertirAplicacionPago(...)`, recalculando `estadoReserva`/`estadoEvento`. Una reserva/evento con saldo > 0 no puede
  quedar `PAGADA`/`PAGADO`.
- `SaldoFavorHandler` → si `saldoFavorGeneradoOriginal > Dinero.CERO`, debita el saldo a favor del cliente y registra un
  `MovimientoSaldoCliente` de tipo `ANULACION_SALDO_A_FAVOR`.
- `AuditoriaHandler` → registra `AuditoriaOperacion` (`tipoOperacion = ANULACION`) con usuario, motivo y operación afectada
  (regla "La auditoría debe conservar quién creó y quién anuló").

**Datos Importantes:**
- La anulación crea una `CancelacionOperacion` (`origenOperacion` = `PAGO`, `motivo`, `usuarioId`, fecha vía `creadoEn`), de
  modo que motivo/usuario/fecha quedan persistidos con identidad propia, igual que en `VentaAnulada`. No se duplica el
  motivo: `MovimientoCaja.motivoAnulacion` se usa para la grilla de caja y `CancelacionOperacion` para la trazabilidad de negocio.
- La anulación no borra el `Pago`: este pasa a `estado = ANULADA`. La etiqueta "REGISTRADO" del Flujo 11 es la presentación
  de `estado = ACTIVA`.
---

## VentaRegistrada
Publicado cuando una venta se registra y se cobra (V1: la venta siempre se cobra en el momento).

**Datos del evento:**

    - Long ventaId;
    - Long clienteId;              ← null si es persona eventual
    - Dinero total;
    - MetodoPago metodoPago;
    - AreaComplejo area;           ← CONFITERIA_CAFETERIA
    - List<DetalleVenta> detalles;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `MovimientoCajaHandler` → genera `MovimientoCaja` de tipo `INGRESO` (área `CONFITERIA_CAFETERIA`).
- `MovimientoStockHandler` → registra `MovimientoStock` de tipo `SALIDA_VENTA` por cada detalle cuyo producto controle stock.
- `ComprobanteHandler` → emite `ComprobanteOperacion` (`TICKET`).
- `AuditoriaHandler` → registra `AuditoriaOperacion`.

**Datos Importantes:**
- Resuelve como una `Venta` impacta caja y stock sin pasar por la entidad `Pago`: la venta es la unica operación cobrable 
  que se salda integramente al crearse.
---

## VentaAnulada
Publicado cuando una venta se anula.

**Datos del evento:**

    - Long ventaId;
    - Long clienteId;              ← null si era persona eventual
    - Dinero total;
    - List<DetalleVenta> detalles;
    - Long usuarioAnulacionId;
    - String motivo;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `MovimientoCajaHandler` → genera `MovimientoCaja` de tipo `ANULACION` referenciando el movimiento original mediante
  `movimientoOriginalAnuladoId`.
- `MovimientoStockHandler` → registra `MovimientoStock` de tipo `DEVOLUCION` restaurando el stock de cada producto.
- `AuditoriaHandler` → registra `AuditoriaOperacion`.

**Datos Importantes:**
- La anulación crea una `CancelacionOperacion` (`origenOperacion` = VENTA, `motivo`, `usuarioId`, fecha vía `creadoEn`). 
  La `Venta` pasa a `estado = ANULADA` (la etiqueta "ANULADA" del Flujo 4 es la presentación de ese estado).
---

## CuotaGenerada
Publicado por cada cuota creada durante `GeneracionCuota`.

**Datos del evento:**

    - Long cuotaId;
    - Long clienteId;
    - Long inscripcionId;
    - PeriodoMensual periodo;
    - Dinero importe;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `NotificacionHandler` (futuro) → puede enviar aviso de vencimiento.
---

## InscripcionActivada
Publicado cuando una inscripción pasa ha estado `ACTIVA`.

**Datos del evento:**

    - Long inscripcionId;
    - Long clienteId;
    - Long actividadId;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- Ninguno en V1. Preparado para futura lógica (Ej.: notificación de bienvenida).
---

## ClienteDadoDeBaja
Publicado cuando un cliente pasa a `INACTIVO`.

**Datos del evento:**

    - Long clienteId;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `InscripcionHandler` → desactiva inscripciones activas del cliente.
---

## OcupacionCreada
Publicado cuando se crea y persiste una nueva `OcupacionEspacio`.

**Datos del evento:**

    - Long ocupacionId;
    - Long espacioFisicoId;
    - HorarioConcreto horario;
    - TipoOcupacion tipoOcupacion;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `AgendaHandler` → actualiza vista de disponibilidad en tiempo real (proyección).
- `NotificacionHandler` (futuro) → puede enviar confirmación al cliente.

**Datos Importantes:**
- La validación de disponibilidad ocurre ANTES de publicar este evento, como precondición en el servicio de
  aplicación. Este evento notifica que la ocupación fue creada exitosamente, no desencadena la validación.
---

## SaldoFavorGenerado
Publicado cuando un pago genera excedente que se registra como saldo a favor del cliente, o cuando se registra un
anticipo puro (cliente sin cuotas pendientes).

**Datos del evento:**

    - Long clienteId;
    - Long pagoId;
    - Dinero montoGenerado;
    - LocalDateTime ocurridoEn;

**Consumidores:**
- `SaldoFavorHandler` → actualiza `SaldoAFavorCliente.montoDisponible` y registra `MovimientoSaldoCliente` de tipo
  `GENERACION_SALDO_A_FAVOR`.
- `AuditoriaHandler` → registra `AuditoriaOperacion`.

**Datos Importantes:**
- Este evento es distinto de `SaldoFavorAplicado`: `SaldoFavorGenerado` ocurre cuando se *acredita* saldo a favor
  (excedente de pago o anticipo); `SaldoFavorAplicado` ocurre cuando se *consume* ese saldo para cubrir una cuota.
- No genera `MovimientoCaja` adicional. El ingreso de caja ya fue registrado por el `PagoRegistrado` que originó este
  excedente.
---

## SaldoFavorAplicado
Publicado cuando se consume saldo a favor para una cuota.

**Datos del evento:**

    - Long clienteId;
    - Long cuotaId;
    - Dinero montoAplicado;
    - LocalDateTime ocurridoEn;

**Datos Importantes:**
- **No genera `MovimientoCaja`**. El saldo a favor es crédito interno; no entra dinero nuevo.
- Sí genera `MovimientoSaldoCliente` (tipo `APLICACION_SALDO_A_CUOTA`).
- Sí genera `AplicacionSaldoFavorCuota` como registro histórico individual de la aplicación.
---
#
#
#
#
#
# 7. ENUMERADOS
#

## Categoría: Monetario
- `Moneda`: `ARS`, `DOLAR`, `EURO`
- `MetodoPago`: `EFECTIVO`, `TRANSFERENCIA`, `DEBITO`, `CREDITO`, `MERCADO_PAGO`, `OTRO`
- `TipoRecargo`: `PORCENTAJE`, `MONTO_FIJO`

## Categoría: Documentación
- `TipoDocumento`: `DNI`, `PASAPORTE`, `CUIT`, `LE`, `LC`
- `TipoComprobante`: `RECIBO`, `TICKET`, `NOTA_CREDITO`
- `EstadoComprobante`: `EMITIDO`, `ANULADO`

## Categoría: Operaciones Generales
- `EstadoOperacion`: `ACTIVA`, `ANULADA`
- `EstadoCobro`: `PENDIENTE`, `PARCIAL`, `SALDADO`, `ANULADO`
- `TipoOrigenOperacion`: `PAGO`, `VENTA`, `RESERVA`, `EVENTO`, `AJUSTE_MANUAL`, `CUOTA`, `SALDO_FAVOR`, `ACTIVIDAD`, `MANTENIMIENTO`, `BLOQUEO_MANUAL`, `INGRESO_MANUAL`, `EGRESO_MANUAL`
- `TipoCobrable`: `CUOTA`, `RESERVA`, `EVENTO`, `VENTA`
- `TipoParticipante`: `CLIENTE_REGISTRADO`, `PERSONA_EVENTUAL`

## Categoría: Usuarios y Seguridad
- `EstadoUsuario`: `ACTIVO`, `INACTIVO`, `BLOQUEADO`
- `EstadoSesion`: `ACTIVA`, `CERRADA`, `EXPIRADA`
- `ModuloSistema`: `CLIENTES`, `INSCRIPCIONES`, `CUOTAS`, `PAGOS`, `VENTAS`, `CAJA`, `RESERVAS`, `EVENTOS`,
  `ACTIVIDADES`, `ESPACIOS`, `OCUPACIONES`, `INFORMES`, `USUARIOS`, `CONFIGURACION`, `BACKUP`

**Nota:** `ESPACIOS` y `OCUPACIONES` soportan la auditoría y los permisos del Flujo 9 (gestión de espacios físicos y
agenda de ocupaciones). `INFORMES` soporta el control de permisos por informe y la auditoría opcional de consultas
sensibles del Flujo 8.
- `PermisoID`: *(uno por operación habilitada, Ej. `CREAR_PAGO`, `ANULAR_VENTA`, `GENERAR_CUOTAS`, etc.)*
- `TipoOperacionAuditoria`: `CREACION`, `MODIFICACION`, `ANULACION`, `LOGIN`, `LOGOUT`, `GENERACION_CUOTAS`,
  `RESTAURACION_BACKUP`, `CONSULTA_INFORME`

**Nota:** `CONSULTA_INFORME` soporta la regla opcional del Flujo 8 ("Toda generación de informe económico sensible podrá
quedar registrada en auditoría si el sistema lo requiere"). Es de uso opcional; permite auditar el acceso a informes
sensibles (Ej.: operaciones anuladas, ingresos) sin forzar el registro de toda consulta.
- `ResultadoAuditoria`: `EXITOSO`, `FALLIDO`, `PARCIAL`
- `TipoEntidadAuditada`: `CLIENTE`, `RESPONSABLE`, `INSCRIPCION`, `ACTIVIDAD`, `ESPACIO_FISICO`, `CUOTA`, `PAGO`, `VENTA`, `PRODUCTO`, `RESERVA`, `EVENTO`, `OCUPACION_ESPACIO`, `MOVIMIENTO_CAJA`, `USUARIO`, `ROL`, `CONFIGURACION`, `BACKUP`, `OTRO`

**Nota:** `ESPACIO_FISICO` Sin este valor la auditoría de espacios debía caer en `OTRO`, perdiendo trazabilidad por tipo de entidad.

## Categoría: Personas y Clientes
- `Genero`: `MASCULINO`, `FEMENINO`, `NO_BINARIO`, `NO_ESPECIFICADO`
- `Parentesco`: `PADRE`, `MADRE`, `TUTOR`, `ABUELO_A`, `TIO_A`, `HERMANO_A`, `OTRO`
- `EstadoRelacion`: `ACTIVA`, `INACTIVA`
- `EstadoCliente`: `ACTIVO`, `INACTIVO`, `SUSPENDIDO`
- `EstadoRegistro`: `ACTIVO`, `INACTIVO`

## Categoría: Actividades e Inscripciones
- `TipoActividad`: `ESCUELA_FUTBOL`, `TAEKWONDO`, `EDUCACION_FISICA`, `ALQUILER_FUTBOL_5`, `SALON_INFANTIL`,
  `CUMPLEANOS_DEPORTIVO`

**Nota de extensibilidad (OCP):** Si el negocio agrega actividades frecuentemente, considerar migrar`TipoActividad` a una
entidad de catálogo configurable en V2, reemplazando el enum por una entidad`CatalogoTipoActividad` administrable desde
la configuración del sistema.

- `EstadoActividad`: `ACTIVA`, `INACTIVA`, `SUSPENDIDA`
- `EstadoInscripcion`: `ACTIVA`, `SUSPENDIDA`, `FINALIZADA`

## Categoría: Cuotas y Saldo
- `EstadoCuota`: `PENDIENTE`, `PARCIAL`, `PAGADA`, `ANULADA`

**Nota:** `PAGADA` representa una cuota con saldo pendiente en cero, cubierta con pago real, con saldo a favor o con
ambos.

- `EstadoVencimiento`: `AL_DIA`, `VENCIDA`, `SIN_DEUDA`

**Nota:** Representa el estado de vencimiento de una cuota de forma independiente a su estado de pago. `AL_DIA` indica
que la cuota tiene saldo pendiente y la fecha de vencimiento no ha pasado. `VENCIDA` indica que la cuota tiene saldo
pendiente y la fecha de vencimiento ya pasó. `SIN_DEUDA` indica que la cuota no tiene saldo pendiente (fue pagada o
anulada). La transición a `VENCIDA` es responsabilidad de un servicio programado. La transición a `SIN_DEUDA` ocurre
cuando `estadoCuota` pasa a `PAGADA` o `ANULADA`.

- `EstadoGeneracionCuota`: `PREVISUALIZACION`, `EN_PROCESO`, `CONFIRMADA`, `FALLIDA`, `ANULADA`

**Nota:** `EN_PROCESO` se asigna al inicio de la transacción atómica para detectar generaciones colgadas y prevenir
ejecuciones simultáneas del mismo período. `ANULADA` permite invalidar una generación confirmada en casos excepcionales
autorizados por el administrador.

- `ResultadoGeneracionCuota`: `GENERADA`, `OMITIDA_DUPLICADO`, `OMITIDA_SIN_PRECIO`, `OMITIDA_INACTIVA`,
  `OMITIDA_INICIO_POSTERIOR`, `OMITIDA_ACTIVIDAD_NO_MENSUAL`, `ERROR`

**Nota:** `OMITIDA_INICIO_POSTERIOR` aplica cuando la `fechaAlta` de la inscripción es posterior al último día del
período generado. Es un caso distinto a `OMITIDA_INACTIVA`.
`OMITIDA_ACTIVIDAD_NO_MENSUAL` aplica cuando una inscripción `ACTIVA` pertenece a una actividad cuyo `generaCuotaMensual`
es `false`. El proceso recorre todas las inscripciones activas y debe poder distinguir este motivo de omisión del resto.
`ERROR` aplica cuando ocurrió una falla técnica o de negocio no contemplada como omisión (Ej.: excepción al calcular la
fecha de vencimiento, precio inválido detectado en tiempo de ejecución, fallo de persistencia parcial). Permite distinguir
entre cuotas deliberadamente omitidas y cuotas que fallaron por error. El campo`motivoOmision` en `DetalleGeneracionCuota`
debe describir el error técnico en estos casos.

- `TipoMovimientoSaldo`: `GENERACION_SALDO_A_FAVOR`, `APLICACION_SALDO_A_CUOTA`, `ANULACION_SALDO_A_FAVOR`, `AJUSTE_MANUAL`

**Nota:** Representa el origen del movimiento, no solo la dirección. `GENERACION_SALDO_A_FAVOR` ocurre cuando un pago
supera la deuda del cliente y genera excedente. `APLICACION_SALDO_A_CUOTA` ocurre cuando se consume saldo a favor para
cubrir una cuota. `ANULACION_SALDO_A_FAVOR` ocurre cuando se anula un pago que había generado saldo a favor: debita el 
saldo previamente acreditado. Se distingue de `AJUSTE_MANUAL` para conservar la trazabilidad del origen (anulación de 
pago vs. corrección manual). `AJUSTE_MANUAL` es una corrección autorizada por el administrador. La dirección del movimiento 
(crédito/débito) se deriva del tipo: `GENERACION_SALDO_A_FAVOR` y `AJUSTE_MANUAL` positivo son créditos; `
APLICACION_SALDO_A_CUOTA`, `ANULACION_SALDO_A_FAVOR` y `AJUSTE_MANUAL` negativo son débitos.

## Categoría: Pagos y Caja
- `EstadoMovimientoCaja`: `ACTIVO`, `ANULADO`

**Nota:** el estado vigente es `ACTIVO` —no `REGISTRADO`— exactamente como lo define el concepto`EstadoMovimientoCaja` 
del Flujo 7 y como se muestra en la columna "Estado" de la grilla de caja diaria. `ANULADO` marca el movimiento original 
cuya anulación generó un `MovimientoCaja` de tipo `ANULACION`.
- `TipoMovimientoCaja`: `INGRESO`, `EGRESO`, `AJUSTE`, `ANULACION`
- `EstadoCajaDiaria`: `ABIERTA`, `CERRADA`
- `AreaComplejo`: `ESCUELA_FUTBOL`, `TAEKWONDO`, `CONFITERIA_CAFETERIA`, `ALQUILER_CANCHA`, `EDUCACION_FISICA`, `CUMPLEANOS_DEPORTIVO`, `SALON_INFANTIL`, `EVENTO_PARTICULAR`, `OTRO`

**Nota (mejora):** `AreaComplejo` es el campo que permite agrupar ingresos por área en la caja diaria y en los informes 
de forma confiable, en lugar de depender del texto libre de `MovimientoCaja.concepto`.
**Derivación desde `TipoEvento`:** `CUMPLEANOS_DEPORTIVO_VARONES` y `CUMPLEANOS_DEPORTIVO_MIXTO` → `CUMPLEANOS_DEPORTIVO`;
`CUMPLEANOS_SALON_INFANTIL` → `SALON_INFANTIL`; `EVENTO_PARTICULAR_INFANTIL` → `EVENTO_PARTICULAR`. `MovimientoCajaHandler`
carga el `Evento` destino (`ReferenciaCobrable.cobradoId`) para resolver el área concreta.
**Derivación desde `TipoReserva`:** `EDUCACION_FISICA_ESCOLAR` → `EDUCACION_FISICA`; resto → `ALQUILER_CANCHA`.
**Derivación desde la actividad en pagos de cuota de una sola actividad:** el `MovimientoCajaHandler`
resuelve el área desde `Cuota.actividadId` → `Actividad.tipo` con el mapeo: `ESCUELA_FUTBOL` → `ESCUELA_FUTBOL`;
`TAEKWONDO` → `TAEKWONDO`; `EDUCACION_FISICA` → `EDUCACION_FISICA`; `ALQUILER_FUTBOL_5` → `ALQUILER_CANCHA`;
`SALON_INFANTIL` → `SALON_INFANTIL`; `CUMPLEANOS_DEPORTIVO` → `CUMPLEANOS_DEPORTIVO`. Cuando un mismo pago salda cuotas de
actividades de distintas áreas, el `MovimientoCaja` lleva `area = OTRO` y la apertura por área se reconstruye desde
`AplicacionPago` → `Cuota` → `Actividad` (ver `MovimientoCaja`).

## Categoría: Espacios y Ocupación
- `TipoEspacio`: `CANCHA_FUTBOL_5`, `SALA_TAEKWONDO`, `SALON_INFANTIL`, `CONFITERIA_CAFETERIA`, `GIMNASIO`, `QUINCHO`, `OTRO`

**Nota:** `CONFITERIA_CAFETERIA` se incorpora porque la pantalla de gestión de espacios del Flujo 9 lista "Confitería / 
cafetería" como espacio administrable. `OTRO` cubre espacios futuros sin atar el enum.

**Nota de extensibilidad (OCP):** Igual que `TipoActividad`, considerar catálogo configurable en V2 si los espacios
físicos cambian frecuentemente.

- `TipoOcupacion`: `RESERVA`, `EVENTO`, `ACTIVIDAD`, `EDUCACION_FISICA`, `MANTENIMIENTO`, `BLOQUEO_MANUAL`, `OTRO`

**Nota:** `RESERVA` representa la ocupación creada por una reserva simple de cancha. El Flujo 5 la denomina `RESERVA_CANCHA`;
se generaliza a `RESERVA` (misma semántica de bloqueo por reserva, sin atar el enum a un único espacio), igual criterio que
`EVENTO`/`ACTIVIDAD`. La trazabilidad fina al origen ya la da `OcupacionEspacio.origenOperacion`.
`EVENTO` generaliza, del mismo modo, los tipos `EVENTO_CANCHA` / `EVENTO_SALON` / `EVENTO_OTRO` que menciona el Flujo 6: el
espacio concreto ya está en `OcupacionEspacio.espacioFisicoId`, por lo que no se atan al enum.
**Mapeo del Flujo 9:** `RESERVA_CANCHA` → `RESERVA`; `EVENTO_CANCHA` / `EVENTO_SALON` → `EVENTO`; `ACTIVIDAD_FIJA` →
`ACTIVIDAD`; `EDUCACION_FISICA_ESCOLAR` → `EDUCACION_FISICA` (se conserva como tipo propio porque el Flujo 9 lo trata como
uso recurrente distinto de una actividad fija común y porque permite filtrarlo en la agenda); `MANTENIMIENTO` y
`BLOQUEO_MANUAL` se mantienen literales; `OTRO` cubre el tipo `OTRO` del Flujo 9.

- `EstadoOcupacion`: `ACTIVA`, `CANCELADA`, `REALIZADA`, `ANULADA`

## Categoría: Reservas y Eventos
- `TipoReserva`: `ALQUILER_FUTBOL_5`, `EDUCACION_FISICA_ESCOLAR`, `RESERVA_DEPORTIVA_COMUN`, `USO_INSTITUCIONAL`, `OTRO`
- `EstadoReserva`: `PENDIENTE`, `SEÑADA`, `PAGADA`, `CANCELADA`, `REALIZADA`
- `TipoEvento`: `CUMPLEANOS_DEPORTIVO_VARONES`, `CUMPLEANOS_DEPORTIVO_MIXTO`, `CUMPLEANOS_SALON_INFANTIL`, `EVENTO_PARTICULAR_INFANTIL`
- `EstadoEvento`: `PENDIENTE`, `SEÑADO`, `PAGADO`, `CANCELADO`, `REALIZADO`
- `TratamientoDineroCancelacion`: `RETENIDO`, `DEVOLUCION_PARCIAL`, `DEVOLUCION_TOTAL`, `REPROGRAMADO`, `A_REVISAR`, `A_SALDO_FAVOR`

**Nota:** unifica el vocabulario del Flujo 5 con el tratamiento monetario de los Flujos 6/11. `RETENIDO` (el complejo conserva
el dinero; reemplaza al antiguo `SIN_DEVOLUCION`). `DEVOLUCION_PARCIAL`/`DEVOLUCION_TOTAL` (= "devuelto manualmente" del
Flujo 5, según el monto; la devolución efectiva se registra por el flujo de devolución/egreso, no borra pagos).
`REPROGRAMADO` (el dinero se traslada a una futura reserva). `A_REVISAR` (decisión administrativa pendiente). `A_SALDO_FAVOR`
queda solo para operaciones donde el negocio lo habilite (cuotas/eventos); NO es válido para reservas en V1 (Flujo 5).

## Categoría: Productos y Ventas
- `EstadoProducto`: `ACTIVO`, `INACTIVO`, `SIN_STOCK`
- `TipoMovimientoStock`: `ENTRADA`, `SALIDA_VENTA`, `AJUSTE_POSITIVO`, `AJUSTE_NEGATIVO`, `DEVOLUCION`, `PERDIDA`

**Nota (mapeo de los tipos de ajuste del Flujo 10, paso 5):** la pantalla "Ajuste de stock" ofrece seis tipos que se
mapean a este enum: `Reposición` → `ENTRADA` (ingreso por compra a proveedor; es también el tipo del `MovimientoStock`
inicial al crear un producto con stock > 0, Flujo 10 paso 15); `Corrección positiva` → `AJUSTE_POSITIVO`;
`Corrección negativa` → `AJUSTE_NEGATIVO`; `Pérdida` → `PERDIDA`; `Devolución` → `DEVOLUCION`; `Otro` → `AJUSTE_POSITIVO`
o `AJUSTE_NEGATIVO` según la dirección del ajuste, con el detalle en `MovimientoStock.observacion`. No se agrega un valor
`OTRO` al enum porque "Otro" no define por sí solo el signo del movimiento (lo determina si suma o resta stock).

## Categoría: Backup
- `TipoBackup`: `MANUAL`, `AUTOMATICO`
- `EstadoBackup`: `EXITOSO`, `FALLIDO`, `INCOMPLETO`
- `FrecuenciaBackup`: `DIARIA`, `SEMANAL`, `MENSUAL`
- `DestinoBackup`: `LOCAL`, `S3`, `FTP`, `GDRIVE`
- `TipoUbicacionBackup`: `LOCAL`, `REMOTO`
- `EstadoRestauracionBackup`: `EN_PROCESO`, `EXITOSA`, `FALLIDA`
- `ResultadoRestauracion`: `COMPLETA`, `PARCIAL`, `FALLIDA`

## Clases auxiliares referenciadas (no son enums)
- `ArchivoBackup`: Value Object / command con `UbicacionBackup ubicacion` + `String checksum`. Se pasa al `ProveedorBackup`.
- `ResultadoBackup`: Value Object devuelto por `ProveedorBackup.crearBackup(...)` con `UbicacionBackup ubicacion`,
  `String checksum`, `EstadoBackup estado` y `String observacion`. El servicio de aplicación lo usa para construir y
  persistir el registro `BackupSistema` (Flujo 13, pasos 7-9).
- `FiltroAuditoria`: Value Object / command de consulta con `LocalDate fechaDesde`, `LocalDate fechaHasta`,
  `Long usuarioId` (null = todos) y `TipoOperacionAuditoria tipoOperacion` (null = todas). Se pasa a
  `AuditoriaConsultaPort.buscar(...)` (Flujo 12, pantalla de consulta de auditoría).
- `PreferenciasSistema`: Clase embeddable con configuraciones booleanas del sistema (Ej. `permitirRegistroSinDNI`,
  `notificarVencimiento`, `aplicarMoraAutomatica`).