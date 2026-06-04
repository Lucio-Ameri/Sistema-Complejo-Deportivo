# Flujo 2 - Apertura mensual, generación de cuotas y aplicación automática de saldo a favor

---
### Objetivo
Permitir que el administrador genere las cuotas mensuales de los clientes inscriptos en actividades activas que tengan
habilitada la generación de cuota mensual, pudiendo definir previamente el precio mensual de cada actividad para el período
correspondiente. Además, el sistema deberá verificar automáticamente si cada cliente posee saldo a favor disponible y aplicarlo
sobre las cuotas recién generadas, dejando trazabilidad de cada aplicación. Este flujo tiene como finalidad automatizar la
generación de cuotas, evitar errores manuales, impedir cuotas duplicadas, conservar el precio histórico utilizado en cada mes,
utilizar correctamente los pagos adelantados realizados por los clientes y registrar formalmente cada proceso mensual para
impedir repeticiones accidentales. La generación de cuotas no debe calcular ingresos reales de caja. Los ingresos reales
se consultan desde los movimientos de caja generados por pagos, ventas, reservas o eventos. La aplicación de saldo a favor
no representa dinero nuevo y, por lo tanto, no debe generar un nuevo movimiento de caja.
---

### Actor Principal
    Administrador del sistema.
---

### Situación inicial
Comienza un nuevo mes y el sistema debe permitir generar las cuotas mensuales correspondientes a las actividades que 
cobran cuota periódica. Para que una actividad participe de este flujo, deberá cumplir estas condiciones:

- Estar activa.
- Tener el atributo generaCuotaMensual = true.
- Tener inscripciones activas asociadas.
- Tener un precio mensual definido o un precio sugerido para el período.

Ejemplos de actividades mensuales:

- Escuela de fútbol.
- Taekwondo.

No todas las actividades generan cuota mensual. No participan de este flujo:

- Alquiler eventual de cancha.
- Cumpleaños deportivos.
- Salón infantil.
- Ventas de confitería/cafetería.
- Eventos particulares.

Estas ultimas 5 operaciones se manejan mediante reservas, eventos, ventas o pagos eventuales.
Algunos clientes pueden tener saldo a favor porque en un mes anterior pagaron más dinero que el necesario para cubrir 
sus cuotas existentes. Ejemplo:

Un cliente debía Mayo por $30.000 y pagó $60.000.
El sistema aplicó:

    - $30.000 a la cuota de Mayo.
    - $30.000 como saldo a favor.

>Cuando se generen las cuotas de junio, el sistema deberá aplicar automáticamente ese saldo a favor sobre la nueva cuota, 
dejando constancia de la aplicación realizada.
---

### Condición para habilitar el flujo
El sistema deberá habilitar la generación de cuotas a partir del primer día hábil del mes. Para la primera versión del 
sistema, esta regla será configurable, pero se usará como valor inicial:

- Primer día hábil del mes.
- Se considerará día hábil de lunes a viernes.
- No se tendrán en cuenta feriados en la primera versión.

Ejemplos:

- Si el día 1 cae lunes, martes, miércoles, jueves o viernes, se habilita ese mismo día.
- Si el día 1 cae sábado, se habilita el lunes siguiente.
- Si el día 1 cae domingo, se habilita el lunes siguiente.

>Más adelante se podrá mejorar este comportamiento incorporando una tabla de feriados o una configuración por actividad.
El sistema no deberá permitir generar cuotas si ya existe una generación mensual confirmada para el mismo período, 
salvo que un usuario administrador realice una acción excepcional claramente identificada, como generar cuotas faltantes 
por nuevas inscripciones.
---

### Aviso en pantalla principal
Cuando corresponda generar las cuotas del mes, el sistema deberá mostrar un aviso en la pantalla principal del administrador.
El aviso deberá mostrarse solamente si se cumplen estas condiciones:

- Ya llegó el primer día hábil del mes o una fecha posterior.
- Las cuotas del mes actual todavía no fueron generadas.
- El usuario tiene permiso de administrador.
- Existen actividades que generan cuota mensual.
- Existen inscripciones activas.

El aviso deberá decir:
- [ "Ya se pueden generar las cuotas del mes actual." ]

El aviso deberá incluir el botón:
- [ Generar cuotas ahora ]

Al presionar el botón "Generar cuotas ahora", el sistema deberá llevar directamente al administrador a la pantalla de
generación mensual de cuotas.
---

### Ejemplo visual del aviso

    Cuotas mensuales disponibles
    Ya se pueden generar las cuotas de Junio 2026.
    Antes de generarlas, podrás revisar o modificar el precio mensual de cada actividad.
    Si algún cliente tiene saldo a favor, el sistema lo aplicará automáticamente a sus nuevas cuotas.

    [Generar cuotas ahora]
---

### Pantalla - Generación mensual de cuotas

    Período: Junio 2026

    Actividades con cuota mensual:
    ----------------------------------------------------------------------------------------------
    Actividad              Genera cuota mensual   Precio sugerido   Precio del mes   Inscriptos activos
    ----------------------------------------------------------------------------------------------
    Escuela de fútbol      Sí                     $30.000           [ $32.000 ]      42
    Taekwondo              Sí                     $26.000           [ $28.000 ]      18
    ----------------------------------------------------------------------------------------------

Importante:

- Solo se mostrarán actividades activas con generaCuotaMensual = true.
- Las actividades eventuales no deberán aparecer en esta pantalla.
- El precio sugerido viene de la actividad o del último precio usado.
- El precio del mes se guardará históricamente en PrecioMensualActividad.

Aplicación de saldo a favor:

    [x] Aplicar automáticamente saldos a favor disponibles

Mensaje aclaratorio visible para el administrador:

    El saldo a favor ya fue cobrado anteriormente. Si se aplica sobre cuotas nuevas, no generará un nuevo ingreso de caja.

    Si el administrador desactiva esta opción, las cuotas se generarán sin aplicar saldo a favor. En ese caso, el saldo
    quedará disponible para futuras aplicaciones manuales o automáticas.

    [Previsualizar cuotas]
    [Cancelar]
---

### Pasos del flujo

    1. El administrador ingresa al sistema.
    2. El sistema verifica que el usuario esté autenticado.
    3. El sistema verifica que el usuario tenga permiso para generar cuotas mensuales.
    4. El sistema carga la pantalla principal.
    5. El sistema verifica la fecha actual.
    6. El sistema calcula si ya llegó el primer día hábil del mes según la configuración vigente.
    7. El sistema verifica si ya existe una generación mensual confirmada para el período actual.
    8. Si todavía no corresponde generar cuotas, el sistema no muestra ningún aviso.
    9. Si las cuotas del mes ya fueron generadas, el sistema no muestra el aviso.
    10. Si corresponde generar cuotas, el sistema muestra el aviso:
        - [ "Ya se pueden generar las cuotas del mes actual." ]

    11. El aviso muestra el botón:
        - [ Generar cuotas ahora ]

    12. El administrador presiona el botón "Generar cuotas ahora".
    13. El sistema redirige al administrador a la pantalla "Generación mensual de cuotas".
    14. El sistema muestra el período que se va a generar. Ejemplo:

        - Junio 2026.

    15. El sistema busca todas las actividades que cumplan:

        - Actividad activa.
        - generaCuotaMensual = true.
        - Actividad disponible para inscripción mensual.

    16. El sistema excluye actividades eventuales como:

        - Alquiler de cancha.
        - Cumpleaños deportivos.
        - Salón infantil.
        - Confitería.
        - Eventos particulares.

    17. El sistema muestra una fila por cada actividad mensual.
    18. Por cada actividad, el sistema muestra:

        - Nombre de la actividad.
        - Indicador de que genera cuota mensual.
        - Precio sugerido.
        - Campo editable para definir el precio del mes.
        - Cantidad de inscripciones activas.
        - Cantidad estimada de cuotas a generar.

    19. El administrador revisa los precios sugeridos.
    20. El administrador puede dejar el precio sugerido o modificarlo. Ejemplo:

        - Escuela de fútbol: $32.000.
        - Taekwondo: $28.000.

    21. El sistema valida que todos los precios ingresados sean mayores a cero.
    22. Si algún precio es cero, negativo o inválido, el sistema muestra un mensaje de error y no permite continuar.
    23. El sistema muestra la opción:
        - [ "Aplicar automáticamente saldos a favor disponibles." ]

    24. Para la primera versión, esta opción estará activada por defecto.
    25. El sistema deberá mostrar una explicación visible indicando que aplicar saldo a favor no genera caja nueva.
    25.1. Si el administrador desactiva la aplicación automática de saldo a favor, la previsualización deberá mostrar el saldo disponible, pero indicar que no será aplicado.
    25.2. Si la opción queda activada, la previsualización deberá mostrar exactamente cuánto saldo se aplicará a cada cuota.
    26. El administrador presiona:
        - [ Previsualizar cuotas ]

    27. El sistema genera una previsualización antes de guardar definitivamente.
    28. Para armar la previsualización, el sistema busca todas las inscripciones activas.
    29. Por cada inscripción activa, el sistema identifica:

        - Cliente.
        - Actividad.
        - Período a generar.
        - Precio mensual definido para la actividad.
        - Fecha de inicio de la inscripción.
        - Día de vencimiento.
        - Fecha exacta de vencimiento.
        - Saldo a favor disponible del cliente, si existe.

    30. El sistema calcula la fecha exacta de vencimiento usando esta regla:

        - Año y mes del período generado.
        - Día de vencimiento configurado en la inscripción.
        - Para la primera versión, el día de vencimiento deberá estar entre 1 y 28.
        - Ejemplo: período Junio 2026 + día 10 = 10/06/2026.

    31. El sistema verifica si la inscripción comenzó dentro del período que se está generando.
    32. Para la primera versión, la regla será:

        - Si la fecha de inicio es anterior o igual al último día del período, la inscripción puede generar cuota.
        - Si la fecha de inicio es posterior al período, no genera cuota.
        - Si la inscripción comenzó a mitad del mes, se genera cuota completa por defecto.
        - La previsualización deberá marcarla como "alta dentro del período" para que el administrador lo vea.
        - Más adelante se podrá agregar cálculo proporcional o política configurable por actividad.

    33. El sistema verifica si ya existe una cuota para:

        - Cliente.
        - Actividad.
        - Período.

    34. Si la cuota ya existe, el sistema la marca como "omitida por duplicado" en la previsualización.
    35. Toda cuota omitida deberá mostrar el motivo de omisión. Ejemplos:

        - Ya existe cuota para cliente, actividad y período.
        - Inscripción no activa.
        - Actividad no genera cuota mensual.
        - Inscripción inicia después del período.

    36. Si la cuota no existe y la inscripción corresponde al período, el sistema la marca como "a generar".
    37. Si el cliente no tiene saldo a favor, la previsualización indica que la cuota quedará con saldo pendiente igual al importe total.
    38. Si el cliente tiene saldo a favor, el sistema calcula cuánto saldo se aplicará automáticamente.
    39. Si el saldo a favor es igual o mayor al importe de la cuota, el sistema indica que la cuota quedará PAGADA.
    40. Si el saldo a favor es menor al importe de la cuota, el sistema indica que la cuota quedará PARCIAL.
    41. Si el saldo a favor del cliente supera el importe de una cuota y el cliente tiene más de una cuota en el mismo período, el sistema aplicará el saldo siguiendo un criterio definido.
    42. El criterio para aplicar saldo a favor dentro de este flujo será:

        - Primero a cuotas nuevas del período actual.
        - Dentro del mismo período, en el orden definido por actividad y luego por inscripción.
        - No se aplicará automáticamente a cuotas antiguas dentro de este flujo, para evitar mezclar generación mensual con gestión de deudas previas.
        - Si se desea aplicar saldo a cuotas antiguas, deberá hacerse en un flujo separado de compensación o cobro.

    43. En la previsualización, el sistema muestra:

        - Período.
        - Actividades incluidas.
        - Precio definido para cada actividad.
        - Cantidad de inscriptos activos por actividad.
        - Cantidad de cuotas que se crearán.
        - Cantidad de cuotas que se omitirán.
        - Motivo de cada omisión.
        - Total bruto esperado a cobrar.
        - Total de saldo a favor que se aplicará.
        - Total neto que quedará pendiente de cobro.
        - Clientes con saldo a favor.
        - Cómo se aplicará el saldo a favor.
        - Inscripciones iniciadas dentro del período.

    44. El administrador revisa la previsualización.
    45. Si detecta un error en los precios, puede volver atrás y corregirlos.
    46. Si todo está correcto, el administrador presiona:
        - [ Confirmar generación ]

    47. El sistema inicia una transacción atómica.
    47.1. El sistema bloquea o controla el proceso para impedir que dos usuarios generen cuotas del mismo período al mismo tiempo.
    48. Dentro de la misma transacción, el sistema crea un registro de GeneracionCuotasMensuales con estado EN_PROCESO.
    49. El sistema guarda el precio mensual definido para cada actividad y período.
    50. El sistema busca todas las inscripciones activas.
    51. Por cada inscripción activa, el sistema identifica la actividad correspondiente.
    52. El sistema verifica que la actividad tenga generaCuotaMensual = true.
    53. El sistema busca el precio mensual definido para esa actividad y ese período.
    54. El sistema calcula la fecha exacta de vencimiento.
    55. El sistema verifica la política de inscripción iniciada dentro del período.
    56. El sistema verifica si ya existe una cuota para:

        - Cliente.
        - Actividad.
        - Período.

    57. La base de datos deberá tener una restricción única lógica para evitar duplicados:

        - cliente_id.
        - actividad_id.
        - periodo.

    58. Si ya existe una cuota para ese cliente, actividad y período, el sistema no la vuelve a crear.
    59. El sistema registra un DetalleGeneracionCuota con estado OMITIDA y motivo "duplicada".
    60. Si no existe, el sistema crea una nueva cuota.
    61. La cuota se crea inicialmente con los siguientes datos:

        - Cliente.
        - Actividad.
        - Período.
        - Fecha de vencimiento.
        - Importe definido para esa actividad en ese mes.
        - Saldo pendiente igual al importe.
        - Estado PENDIENTE.
        - Referencia a la generación mensual que la creó.

    62. Luego de crear la cuota, el sistema verifica si la opción de aplicar saldo a favor está activada.
    63. Si la opción está desactivada, no se aplica saldo a favor aunque el cliente posea saldo disponible. La cuota queda como fue generada y el saldo del cliente permanece disponible.
    64. Si la opción está activada, el sistema verifica si el cliente posee saldo a favor disponible.
    65. Si el cliente no posee saldo a favor, la cuota queda como fue generada:

        - Importe total.
        - Saldo pendiente igual al importe.
        - Estado PENDIENTE.

    66. Si el cliente posee saldo a favor, el sistema calcula el monto a aplicar.
    67. El monto a aplicar será el menor entre:

        - Saldo a favor disponible del cliente.
        - Saldo pendiente de la cuota.

    68. El sistema aplica automáticamente ese monto a la cuota.
    69. El sistema descuenta el monto aplicado del saldo a favor del cliente.
    70. El sistema descuenta el monto aplicado del saldo pendiente de la cuota.
    71. Si el saldo pendiente de la cuota queda en cero, la cuota queda en estado PAGADA.
    72. Si el saldo pendiente de la cuota queda mayor a cero, la cuota queda en estado PARCIAL.
    73. El sistema registra una AplicacionSaldoFavorCuota.
    74. La aplicación de saldo a favor debe guardar:

        - Cliente.
        - Cuota.
        - Monto aplicado.
        - Fecha y hora de aplicación.
        - Motivo.
        - Usuario o proceso que realizó la aplicación.
        - Referencia al movimiento de saldo que redujo el saldo a favor.

    75. El sistema registra un MovimientoSaldoCliente de tipo APLICACION_SALDO_A_CUOTA.
    76. El movimiento de saldo deberá afectar el saldo a favor del cliente, pero no deberá afectar caja.
    77. La aplicación de saldo a favor no debe generar un nuevo movimiento de caja.
    78. El dinero ya fue registrado en caja cuando el cliente realizó el pago original que generó el saldo a favor.
    79. Por cada cuota creada u omitida, el sistema registra un DetalleGeneracionCuota.
    80. Una vez procesadas todas las inscripciones activas, el sistema actualiza GeneracionCuotasMensuales con estado CONFIRMADA.
    81. Si ocurre cualquier error en la generación de cuotas, precios, saldos o detalles, el sistema revierte toda la transacción.
    82. Si la transacción falla, no debe quedar una generación parcial inconsistente.
    83. Una vez procesadas todas las inscripciones activas, el sistema muestra un resumen final.
    84. El resumen final deberá indicar:

        - Período generado.
        - Estado de la generación.
        - Cantidad total de cuotas generadas.
        - Cantidad total de cuotas omitidas.
        - Motivos de omisión.
        - Total bruto generado.
        - Total de saldo a favor aplicado.
        - Total pendiente real de cobro.
        - Cantidad de cuotas generadas como PENDIENTE.
        - Cantidad de cuotas generadas como PARCIAL.
        - Cantidad de cuotas generadas como PAGADA por saldo a favor.
        - Precio utilizado para cada actividad.
        - Fecha y hora de generación.
        - Usuario que confirmó la generación.
        - Resultado de auditoría de la generación.

    85. Luego de generar las cuotas, el aviso de la pantalla principal ya no deberá mostrarse para ese período.
---

### Subflujo - Generar cuotas faltantes del período
Este subflujo se usará cuando el período ya tenga una GeneracionCuotasMensuales CONFIRMADA, pero luego se carguen nuevas
inscripciones que también necesitan cuota para ese mismo período.

    1. El administrador ingresa al módulo de cuotas.
    2. Selecciona un período ya generado.
    3. El sistema detecta que ya existe una GeneracionCuotasMensuales confirmada.
    4. El sistema ofrece la opción:
        - [Generar cuotas faltantes]

    5. El sistema busca inscripciones activas que:
        - correspondan a actividades con generaCuotaMensual = true;
        - correspondan al período seleccionado;
        - no tengan cuota para cliente, actividad y período;
        - no inicien después del período.

    6. El sistema usa el PrecioMensualActividad ya definido para ese período.
    7. El sistema no permite modificar libremente el precio mensual ya usado en cuotas existentes.
    8. El sistema previsualiza únicamente las cuotas faltantes.
    9. El administrador confirma.
    10. El sistema crea solo las cuotas faltantes.
    11. El sistema registra DetalleGeneracionCuota con motivo "generación faltante".
    12. El sistema registra auditoría con usuario, fecha, hora, período y resultado.
    13. El sistema muestra resumen final de cuotas faltantes generadas.
---

### Pantalla - Previsualización de cuotas - Junio 2026

    Actividades:

        Escuela de fútbol:

            - Precio del mes: $32.000
            - Inscriptos activos: 42
            - Cuotas a generar: 42
            - Cuotas omitidas por ya existir: 0
            - Total bruto: $1.344.000

        Taekwondo:

            - Precio del mes: $28.000
            - Inscriptos activos: 18
            - Cuotas a generar: 18
            - Cuotas omitidas por ya existir: 0
            - Total bruto: $504.000

    Cuotas omitidas:

        ---------------------------------------------------------------------------------------------
        Cliente          Actividad            Período       Motivo
        ---------------------------------------------------------------------------------------------
        Pedro López      Escuela de fútbol    Junio 2026    Ya existe cuota para cliente, actividad y período
        Ana Torres       Taekwondo            Junio 2026    Inscripción inicia después del período
        ---------------------------------------------------------------------------------------------

    Inscripciones iniciadas dentro del período:

        ---------------------------------------------------------------------------------------------
        Cliente          Actividad            Fecha inicio      Tratamiento
        ---------------------------------------------------------------------------------------------
        Bruno Castro     Escuela de fútbol    15/06/2026        Genera cuota completa en versión 1
        ---------------------------------------------------------------------------------------------

    Aplicación automática de saldos a favor:
        -----------------------------------------------------------------------------------------------------------------
        Cliente          Actividad            Cuota     Saldo a favor previo   Se aplicará   Saldo pendiente cuota   Saldo a favor restante
        -----------------------------------------------------------------------------------------------------------------
        Mateo Gómez      Escuela de fútbol    $32.000   $30.000                $30.000       $2.000                  $0
        Lucas Pérez      Taekwondo            $28.000   $28.000                $28.000       $0                      $0
        Sofía Díaz       Escuela de fútbol    $32.000   $40.000                $32.000       $0                      $8.000
        -----------------------------------------------------------------------------------------------------------------

Aclaración:

- "Saldo pendiente cuota" indica cuánto quedará debiendo el cliente por esa cuota.
- "Saldo a favor restante" indica cuánto dinero seguirá disponible para futuras cuotas.
- La aplicación de saldo a favor no representa ingreso nuevo de caja.


    Resumen general:

        Total bruto generado:              $1.848.000
        Total saldo a favor aplicado:      $90.000
        Total pendiente real de cobro:     $1.758.000

        Cuotas que quedarán PAGADAS por saldo a favor:   2
        Cuotas que quedarán PARCIALES por saldo a favor: 1
        Cuotas que quedarán PENDIENTES:                  57

    [Confirmar generación]
    [Volver a editar precios]
    [Cancelar]
---

### Ejemplo 1 - cliente sin saldo a favor

    Cliente: Juan Pérez
    Actividad: Escuela de fútbol
    Período: Junio 2026
    Precio mensual: $32.000
    Saldo a favor: $0

    Resultado:

        - Se genera cuota por $32.000.
        - Saldo pendiente: $32.000.
        - Estado: PENDIENTE.
        - No se aplica saldo a favor.
---

### Ejemplo 2 - saldo a favor menor que la cuota

    Cliente: Mateo Gómez
    Actividad: Escuela de fútbol
    Período: Junio 2026
    Precio mensual: $32.000
    Saldo a favor disponible: $30.000

    Resultado:

        - Se genera cuota por $32.000.
        - Se aplica saldo a favor por $30.000.
        - Saldo pendiente de cuota: $2.000.
        - Estado final de cuota: PARCIAL.
        - Saldo a favor restante: $0.
        - No se registra movimiento de caja nuevo.
---

### Ejemplo 3 - saldo a favor igual a la cuota

    Cliente: Lucas Pérez
    Actividad: Taekwondo
    Período: Junio 2026
    Precio mensual: $28.000
    Saldo a favor disponible: $28.000

    Resultado:

        - Se genera cuota por $28.000.
        - Se aplica saldo a favor por $28.000.
        - Saldo pendiente de cuota: $0.
        - Estado final de cuota: PAGADA.
        - Saldo a favor restante: $0.
        - No se registra movimiento de caja nuevo.
---

### Ejemplo 4 - saldo a favor mayor que la cuota

    Cliente: Sofía Díaz
    Actividad: Escuela de fútbol
    Período: Junio 2026
    Precio mensual: $32.000
    Saldo a favor disponible: $40.000

    Resultado:

        - Se genera cuota por $32.000.
        - Se aplica saldo a favor por $32.000.
        - Saldo pendiente de cuota: $0.
        - Estado final de cuota: PAGADA.
        - Saldo a favor restante: $8.000.
        - Los $8.000 restantes quedan disponibles para futuras cuotas.
        - No se registra movimiento de caja nuevo.
---

### Ejemplo 5 - cliente con dos actividades y saldo a favor

    Cliente: Tomás Rodríguez
    Saldo a favor disponible: $50.000

    Cuotas generadas en Junio 2026:

        - Escuela de fútbol: $32.000.
        - Taekwondo: $28.000.

    Total de cuotas: $60.000.

    Aplicación automática:

        1. Se aplica saldo a Escuela de fútbol:
           - Cuota: $32.000.
           - Saldo aplicado: $32.000.
           - Estado: PAGADA.

        2. Queda saldo a favor disponible:
           - $18.000.

        3. Se aplica saldo restante a Taekwondo:
           - Cuota: $28.000.
           - Saldo aplicado: $18.000.
           - Saldo pendiente: $10.000.
           - Estado: PARCIAL.

    Resultado final:

        - Escuela de fútbol: PAGADA.
        - Taekwondo: PARCIAL.
        - Saldo a favor restante: $0.
        - No se registra movimiento de caja nuevo.
---

### Decisiones importantes

- ¿El usuario está autenticado?
- ¿El usuario tiene permiso para generar cuotas?
- ¿Ya llegó el primer día hábil del mes según la configuración vigente?
- ¿Las cuotas del mes ya fueron generadas?
- ¿Existe una GeneracionCuotasMensuales confirmada para el período?
- ¿Existen actividades activas?
- ¿Existen actividades con generaCuotaMensual = true?
- ¿Existen inscripciones activas?
- ¿Qué actividades deben generar cuota mensual?
- ¿Qué precio tendrá cada actividad en este período?
- ¿Todos los precios ingresados son válidos?
- ¿Cómo se calculará la fecha exacta de vencimiento?
- ¿La inscripción comenzó antes o durante el período?
- ¿La inscripción comenzó a mitad de mes?
- ¿Se debe generar cuota completa, proporcional o no generar?
- ¿El administrador confirma la previsualización?
- ¿Ya existe una cuota para el mismo cliente, actividad y período?
- ¿Por qué motivo se omite una cuota?
- ¿El cliente tiene saldo a favor?
- ¿Cuánto saldo a favor se puede aplicar a cada cuota?
- ¿El saldo a favor cubre toda la cuota o solo una parte?
- ¿Queda saldo a favor restante para el cliente?
- ¿La cuota queda pendiente, parcial o pagada?
- ¿La operación completa puede confirmarse dentro de una única transacción?
---

### Datos que intervienen

- Actividad.
- PrecioMensualActividad.
- Cliente.
- Inscripción.
- Cuota.
- SaldoAFavorCliente.
- AplicacionSaldoFavorCuota.
- MovimientoSaldoCliente.
- GeneracionCuotasMensuales.
- DetalleGeneracionCuota.
- Usuario administrador.
- Auditoria.
---

### Nuevo concepto detectado

- [ PrecioMensualActividad ]

Este concepto representa el precio que tendrá una actividad determinada en un mes específico. Ejemplo:

    - Escuela de fútbol - Junio 2026 - $32.000.
    - Taekwondo - Junio 2026 - $28.000.
    - Escuela de fútbol - Julio 2026 - $35.000.

>Este precio mensual no debe modificar cuotas de meses anteriores.
---

### Nuevo concepto detectado

- [ SaldoAFavorCliente ]

Este concepto representa dinero que el cliente ya pagó, pero que todavía no fue aplicado totalmente a una cuota.
Se genera normalmente cuando el cliente paga más dinero que el necesario para cubrir las cuotas seleccionadas. Ejemplo:

El cliente debía Mayo por $30.000 y pagó $60.000. Resultado:

        - $30.000 se aplican a Mayo.
        - $30.000 quedan como saldo a favor.

>Ese saldo a favor podrá aplicarse automáticamente a cuotas futuras.
---

### Nuevo concepto detectado

- [ AplicacionSaldoFavorCuota ]

Este concepto representa cuánto saldo a favor fue aplicado a una cuota determinada.
Es necesario para conservar historial y trazabilidad. Ejemplo:

    Cliente: Mateo Gómez
    Cuota: Junio 2026 - Escuela de fútbol
    Monto aplicado: $30.000
    Fecha de aplicación: 01/06/2026
    Motivo: Aplicación automática de saldo a favor.

>Importante: La aplicación de saldo a favor no representa dinero nuevo que entra al negocio. Por lo tanto, no debe 
generar un nuevo movimiento de caja.
---

### Nuevo concepto detectado

- [ GeneracionCuotasMensuales ]

Este concepto representa el proceso mensual confirmado de generación de cuotas para un período determinado. Debe guardar:

- Período generado.
- Fecha y hora de inicio.
- Fecha y hora de confirmación.
- Usuario que confirmó.
- Estado: BORRADOR/PREVISUALIZADA, EN_PROCESO, CONFIRMADA, FALLIDA, ANULADA.
- Total de cuotas generadas.
- Total de cuotas omitidas.
- Total bruto.
- Total de saldo a favor aplicado.
- Total pendiente final.

>Este concepto evita generar dos veces accidentalmente las cuotas del mismo mes.
---

### Nuevo concepto detectado

- [ DetalleGeneracionCuota ]

Este concepto representa el resultado individual de cada inscripción procesada durante la generación mensual. Debe guardar:

- Generación mensual asociada.
- Cliente.
- Actividad.
- Inscripción.
- Cuota generada, si corresponde.
- Estado del detalle: GENERADA, OMITIDA, ERROR.
- Motivo de omisión o error.
- Importe previsto.
- Saldo a favor aplicado.
- Saldo pendiente final.

>Este concepto permite auditar por qué se creó u omitió cada cuota.
---

### Nuevo concepto detectado

- [ MovimientoSaldoCliente ]

Este concepto representa movimientos que aumentan o disminuyen el saldo a favor del cliente. Ejemplos:

    - GENERACION_SALDO_A_FAVOR: cuando un pago supera la deuda seleccionada.
    - APLICACION_SALDO_A_CUOTA: cuando se usa saldo a favor para cubrir una cuota.
    - AJUSTE_MANUAL: cuando un administrador corrige saldo por autorización.

>Importante: No todo movimiento de saldo genera caja. Solo el pago original que generó saldo a favor genera movimiento 
de caja.
---

### Reglas de negocio detectadas

- Las cuotas mensuales solo podrán generarse a partir del primer día hábil del mes según configuración.
- La regla de primer día hábil deberá ser configurable.
- El sistema deberá mostrar un aviso en la pantalla principal cuando las cuotas del mes puedan generarse.
- El aviso deberá incluir el botón "Generar cuotas ahora".
- El botón deberá llevar directamente a la pantalla de generación mensual de cuotas.
- El aviso no deberá mostrarse si las cuotas del mes ya fueron generadas.
- El aviso no deberá mostrarse a usuarios sin permiso para generar cuotas.
- La generación mensual deberá registrarse en GeneracionCuotasMensuales.
- No se podrá confirmar dos veces una generación mensual para el mismo período.
- La generación de cuotas deberá dividirse por actividad.
- Solo deberán incluirse actividades activas con generaCuotaMensual = true.
- Cada actividad que genere cuota mensual deberá tener un precio definido para el período.
- El administrador podrá modificar el precio mensual de cada actividad antes de generar las cuotas.
- El precio definido para una actividad en un mes deberá quedar guardado históricamente.
- Cambiar el precio de un mes no deberá modificar cuotas de meses anteriores.
- Las cuotas se generarán solo para inscripciones activas.
- Una inscripción activa generará una cuota para su actividad correspondiente, salvo que sea omitida por una regla definida.
- Un cliente inscripto en dos actividades podrá recibir dos cuotas diferentes en el mismo mes.
- No se podrá generar una cuota duplicada para el mismo cliente, actividad y período.
- Deberá existir una restricción única lógica cliente + actividad + período.
- Toda cuota omitida deberá registrar el motivo de omisión.
- La fecha de vencimiento se calculará usando el período generado y el día de vencimiento de la inscripción.
- Para la primera versión, el día de vencimiento deberá estar entre 1 y 28.
- Si una inscripción inicia después del período generado, no deberá generar cuota.
- Si una inscripción inicia dentro del período generado, en la primera versión generará cuota completa.
- Las inscripciones iniciadas dentro del período deberán mostrarse claramente en la previsualización.
- Una cuota nueva deberá crearse inicialmente con saldo pendiente igual al importe.
- Una cuota nueva deberá crearse inicialmente en estado PENDIENTE.
- No se podrán generar cuotas si alguna actividad mensual tiene precio cero, negativo o inválido.
- El sistema deberá mostrar una previsualización antes de confirmar la generación definitiva.
- La previsualización deberá mostrar qué clientes tienen saldo a favor y cómo se aplicará.
- La aplicación automática de saldo a favor deberá estar visible y explicada para el administrador.
- Al generar cuotas mensuales, el sistema deberá verificar si el cliente posee saldo a favor.
- El saldo a favor deberá aplicarse automáticamente a cuotas recién generadas si la opción está activada.
- El saldo a favor se aplicará únicamente sobre cuotas del período actual dentro de este flujo.
- Si se desea aplicar saldo a cuotas antiguas, deberá hacerse en un flujo separado.
- El monto aplicado será como máximo el saldo pendiente de la cuota.
- Si el saldo a favor cubre el total de la cuota, la cuota quedará PAGADA.
- Si el saldo a favor cubre solo una parte de la cuota, la cuota quedará PARCIAL.
- Si el saldo a favor es mayor al importe de la cuota, el sobrante seguirá disponible para futuras cuotas.
- La aplicación del saldo a favor debe quedar registrada históricamente.
- La aplicación del saldo a favor deberá generar un MovimientoSaldoCliente.
- La aplicación del saldo a favor no debe generar un nuevo movimiento de caja.
- El dinero del saldo a favor ya fue registrado en caja cuando se recibió originalmente.
- La generación de cuotas y la aplicación de saldo deberán ejecutarse en una única transacción.
- Si falla una parte del proceso, no deberá quedar una generación parcial confirmada.
- El resumen final deberá mostrar cuánto saldo a favor fue aplicado.
- El resumen final deberá mostrar cuotas generadas, omitidas, pendientes, parciales y pagadas.
- Toda generación mensual confirmada deberá registrar auditoría con usuario, fecha, hora, período generado y resultado del proceso.
- Si el administrador desactiva la aplicación automática de saldo a favor, las cuotas se generarán sin aplicar saldo y el saldo quedará disponible.
- Si ya existe PrecioMensualActividad para un período con cuotas generadas, no deberá modificarse libremente desde este flujo.
- En una generación de cuotas faltantes, el sistema deberá usar el PrecioMensualActividad ya definido para ese período.
- El sistema deberá impedir que dos usuarios generen cuotas del mismo período al mismo tiempo.
- La cantidad estimada de cuotas a generar deberá excluir cuotas ya existentes e inscripciones que no correspondan al período.
- Si se genera una cuota faltante luego de una generación mensual confirmada, deberá quedar registrada como generación complementaria o detalle adicional.
---

### Resultado final
El sistema genera las cuotas mensuales del período seleccionado usando el precio definido por el administrador para cada 
actividad mensual activa. Cada cuota queda asociada al cliente, a la actividad y al mes correspondiente. El importe de 
cada cuota queda guardado históricamente, por lo que futuros aumentos no modifican cuotas anteriores.
La generación queda registrada formalmente en GeneracionCuotasMensuales, junto con sus detalles individuales en 
DetalleGeneracionCuota. Esto permite saber qué cuotas se generaron, cuáles se omitieron y por qué motivo.
Luego de generar cada cuota, el sistema verifica automáticamente si el cliente posee saldo a favor disponible. Si existe 
saldo a favor y la opción está activada, el sistema lo aplica automáticamente sobre la cuota recién generada.
Según el monto aplicado, la cuota puede quedar:

    - PENDIENTE, si no se aplicó saldo a favor.
    - PARCIAL, si el saldo a favor cubrió solo una parte.
    - PAGADA, si el saldo a favor cubrió el total.

La aplicación del saldo a favor queda registrada como historial mediante AplicacionSaldoFavorCuota y MovimientoSaldoCliente, 
pero no genera un nuevo movimiento de caja, porque el dinero ya ingresó al momento del pago original.
El sistema evita crear cuotas duplicadas mediante validación lógica y restricción única por cliente, actividad y período. 
Además, todo el proceso se ejecuta en una única transacción para evitar generaciones incompletas o inconsistentes. La 
generación confirmada queda auditada y protegida contra dobles ejecuciones simultáneas.

![Diagrama de flujo](./imagenes-de-los-flujos/flujo-2.png)
