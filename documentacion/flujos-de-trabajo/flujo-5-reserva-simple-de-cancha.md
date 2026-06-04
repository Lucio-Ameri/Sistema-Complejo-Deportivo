# Flujo 5 - Reserva simple de cancha

---
## Objetivo
Permitir que el administrador registre reservas simples de la cancha de fútbol 5, verificando automáticamente la
disponibilidad real del espacio contra cualquier ocupación existente. Este flujo tiene como finalidad reemplazar la
agenda manual de alquileres de cancha y permitir controlar usos deportivos comunes, educación física escolar, reservas
institucionales o alquileres simples de la cancha.

Este flujo **no debe utilizarse para cumpleaños deportivos ni eventos infantiles**. Los cumpleaños deportivos y eventos
infantiles deberán registrarse desde el **Flujo 6 - Cumpleaños y eventos**.

> Corrección de alcance: aunque inicialmente pudiera pensarse que un cumpleaños deportivo es una reserva de cancha, 
dentro del sistema deberá tratarse como **evento**, no como reserva simple. Las reservas simples quedan reservadas para
alquileres comunes, educación física escolar, usos institucionales o bloqueos deportivos no festivos.
---

## Actor principal
    Administrador del sistema.
---

## Situación inicial
Una persona, grupo o institución solicita reservar la cancha de fútbol 5 para un uso simple. La reserva puede ser para:

- Alquiler de cancha fútbol 5.
- Educación física escolar.
- Reserva deportiva común.
- Uso institucional de la cancha.
- Otro alquiler simple de la cancha.

La persona que reserva puede ser:

- Cliente registrado.
- Persona eventual.
- Representante de una escuela.
- Responsable de un grupo deportivo.
---

# Casos que NO corresponden a este flujo
No deben registrarse en este flujo:

- Cumpleaños deportivo varones.
- Cumpleaños deportivo mixto.
- Cumpleaños en salón infantil.
- Eventos particulares infantiles.
- Fiestas infantiles.

Esos casos corresponden al **Flujo 6 - Cumpleaños y eventos**.

## Corrección obligatoria al alcance general
En el alcance del sistema, la sección de reservas deberá quedar definida así:

    Tipos de reserva simple:
        
        - Alquiler fútbol 5.
        - Educación física escolar.
        - Reserva deportiva común.
        - Uso institucional.
        - Otro alquiler simple.
    
    No incluir como reserva simple:
        
        - Cumpleaños deportivo.
        - Cumpleaños en salón infantil.
        - Eventos infantiles.


>Los cumpleaños deportivos deberán figurar únicamente dentro del módulo de eventos.
---

## Condición para iniciar el flujo
Para iniciar este flujo deben cumplirse estas condiciones:

- El administrador debe estar autenticado.
- El administrador debe tener permiso para registrar reservas.
- El sistema debe tener configurado el espacio físico `Cancha de fútbol 5`.
- El sistema debe permitir consultar disponibilidad por fecha y horario.
- Deben existir tipos de reserva simple configurados.
- En la primera versión, el saldo a favor del cliente no se aplicará a reservas de cancha.

>Las reservas deberán cobrarse mediante pagos reales. Si una reserva recibe seña o pago total, ese dinero deberá generar
movimiento de caja.
---

# Concepto central - disponibilidad de espacios
Antes de registrar cualquier reserva simple de cancha, el sistema deberá verificar si la cancha está disponible.

La disponibilidad no debe revisarse solamente contra otras reservas simples. El sistema debe verificar la cancha contra
cualquier ocupación activa del mismo espacio, incluyendo:

- Reservas simples de cancha.
- Cumpleaños deportivos.
- Escuela de fútbol.
- Educación física escolar.
- Eventos deportivos.
- Bloqueos manuales.
- Mantenimiento.
- Otros usos futuros de la cancha.

Esto significa que el control real de disponibilidad deberá hacerse contra una entidad transversal llamada
`OcupacionEspacio`.
---

## Concepto transversal - OcupacionEspacio
Toda operación que utilice un espacio físico del complejo debe crear una ocupación de espacio.

En este flujo, al confirmar una reserva simple de cancha, el sistema deberá crear una `OcupacionEspacio` asociada a la
reserva. Esa ocupación será la que bloquee la cancha para futuras reservas, eventos o actividades en el mismo horario.

Este concepto deberá pasar al alcance general o glosario del sistema porque será usado por varios módulos:

- Reservas simples.
- Eventos.
- Cumpleaños deportivos.
- Actividades recurrentes.
- Educación física escolar.
- Mantenimiento.
- Bloqueos manuales.
---

## Datos principales de OcupacionEspacio

- Espacio utilizado.
- Fecha.
- Hora de inicio.
- Hora de fin.
- Tipo de ocupación.
- Referencia al origen.
- Estado.
- Observación.
- Usuario que creó la ocupación.
- Fecha y hora de creación.
---

## Espacio utilizado en este flujo

- Cancha de fútbol 5.
---

## Tipo de ocupación generado por este flujo

- `RESERVA_CANCHA`.
---

## Estados posibles de OcupacionEspacio

- `ACTIVA`: bloquea el espacio.
- `CANCELADA`: ya no bloquea el espacio, pero queda como historial.
- `REALIZADA`: la ocupación ocurrió.
- `ANULADA`: se anuló por error administrativo.

>Solo las ocupaciones en estado `ACTIVA` deberán bloquear disponibilidad.
---

## Regla de superposición horaria
Para detectar conflicto, el sistema deberá considerar que dos ocupaciones se superponen cuando:
    
    nuevaHoraInicio < ocupacionExistenteHoraFin y nuevaHoraFin > ocupacionExistenteHoraInicio

Ejemplo con conflicto:

    Ocupación existente:
        18:00 a 19:00
    
    Nueva reserva:
        18:30 a 20:00
    
    Resultado:
        Hay conflicto.


Ejemplo sin conflicto:

    Ocupación existente:
        18:00 a 19:00
    
    Nueva reserva:
        19:00 a 20:00
    
    Resultado:
        No hay conflicto.
---

## Regla sobre duración y fecha de la reserva
Para la primera versión, una reserva simple de cancha no podrá cruzar de un día a otro. Reglas:

- La fecha de inicio y fin será la misma.
- La hora de inicio deberá ser anterior a la hora de fin.
- Si se necesita bloquear un espacio por más de un día, deberá registrarse mediante un flujo específico de bloqueos, mantenimiento o eventos especiales.
- La duración mínima de una reserva podrá ser configurable.
- Para la primera versión, se recomienda no permitir reservas de duración cero ni extremadamente cortas.

Ejemplo no permitido en versión 1:

    Fecha: 10/06/2026
    Hora inicio: 23:00
    Hora fin: 01:00
---

## Pantalla - Nueva reserva simple de cancha

    Nueva reserva simple de cancha
    
    Fecha:              [ 15/05/2026        ]
    Hora inicio:        [ 18:00             ]
    Hora fin:           [ 19:00             ]
    
    Tipo de reserva:    [ Alquiler fútbol 5 ]
    
    Responsable:        [ Juan Pérez        ]
    Teléfono:           [ 11-5555-5555      ]
    
    Cliente registrado: [ No                ]
    
    Importe total:      [ $35.000           ]
    
    Tipo de pago inicial:
    ( ) Sin pago inicial
    (x) Seña
    ( ) Pago total
    
    Monto pagado:       [ $10.000           ]
    Método de pago:     [ Transferencia     ]
    
    Observaciones:      [                   ]
    
    [Verificar disponibilidad]
    [Guardar reserva]
    [Cancelar]
---

## Regla sobre pago inicial
La pantalla deberá permitir diferenciar claramente entre:

- Sin pago inicial.
- Seña.
- Pago total.

Reglas:

- Si no hay pago inicial, la reserva queda `PENDIENTE`.
- Si el monto pagado inicialmente es menor al importe total, la reserva queda `SEÑADA`.
- Si el monto pagado inicialmente es igual al importe total, la reserva queda `PAGADA`.
- El monto pagado inicialmente no podrá superar el importe total.
- Todo pago inicial deberá generar `PagoReserva` y `MovimientoCaja` de tipo `INGRESO`.
---

## Pasos del flujo principal

    1. El administrador ingresa al sistema.
    2. El administrador accede al módulo `Reservas`.
    3. El sistema muestra la agenda de la cancha.
    4. El administrador selecciona la opción:

        - [ Nueva reserva de cancha ]


    5. El sistema muestra el formulario de reserva simple de cancha.
    6. El administrador ingresa la fecha solicitada.
    7. El administrador ingresa la hora de inicio.
    8. El administrador ingresa la hora de fin.
    9. El administrador selecciona el tipo de reserva:

        - Alquiler fútbol 5.
        - Educación física escolar.
        - Reserva deportiva común.
        - Uso institucional.
        - Otro alquiler simple.

    10. El administrador carga los datos del responsable:

        - Nombre.
        - Teléfono.

    11. El administrador indica si la persona está asociada a un cliente registrado.
    12. Si está asociada a un cliente registrado, el administrador busca y selecciona el cliente.
    13. Si es una persona eventual, el sistema permite continuar con nombre y teléfono del responsable.
    14. El sistema valida que la fecha sea válida.
    15. Si la fecha es anterior a la fecha actual, el sistema deberá pedir confirmación o impedir la operación según configuración.
    16. El sistema valida que la hora de inicio sea anterior a la hora de fin.
    17. El sistema identifica el espacio que se quiere utilizar. En este flujo, el espacio será:

        - Cancha de fútbol 5

    18. El sistema ejecuta la verificación automática de disponibilidad.
    19. Para verificar disponibilidad de la cancha, el sistema busca todas las ocupaciones activas de ese espacio en la fecha seleccionada.
    20. Las ocupaciones que pueden bloquear la cancha son:

        - Reservas simples de cancha.
        - Cumpleaños deportivos.
        - Escuela de fútbol.
        - Educación física escolar.
        - Eventos deportivos.
        - Bloqueos manuales.
        - Mantenimiento.
        - Otros usos futuros de la cancha.

    21. El sistema compara el horario solicitado contra cada ocupación existente.
    22. Si existe una ocupación activa con horario superpuesto, el sistema no permite continuar.
    23. El sistema muestra un mensaje claro indicando qué ocupa la cancha. Ejemplo:
        
        La cancha no está disponible en ese horario.

            Ocupación existente:
            Escuela de fútbol
            Horario: 17:00 a 19:00.

    24. Si no existe conflicto, el sistema permite continuar con la carga de importe, seña y datos económicos.
    25. El administrador ingresa el importe total de la reserva.
    26. El sistema valida que el importe total sea mayor a cero.
    27. El administrador indica el tipo de pago inicial: sin pago, seña o pago total.
    28. Si no hay pago inicial, la reserva queda como `PENDIENTE`.
    29. Si hay seña o pago total, el administrador ingresa el monto pagado.
    30. El sistema valida que el monto pagado sea mayor a cero cuando exista pago inicial.
    31. El sistema valida que el monto pagado no supere el importe total.
    32. Si hay seña, el administrador selecciona el método de pago.
    33. El sistema calcula el saldo pendiente.
    34. El sistema muestra una previsualización de la reserva.
    35. La previsualización muestra:

        - Fecha.
        - Horario.
        - Tipo de reserva.
        - Responsable.
        - Teléfono.
        - Cliente asociado, si corresponde.
        - Importe total.
        - Seña.
        - Saldo pendiente.
        - Método de pago, si corresponde.
        - Estado inicial.
        - Espacio a bloquear.
        - Aclaración de que el saldo a favor del cliente no se aplica a reservas.

    36. El administrador revisa la previsualización.
    37. Si detecta un error, puede volver atrás y corregir.
    38. Si todo está correcto, el administrador confirma la reserva.
    39. El sistema ejecuta la operación en una única transacción.
    40. Dentro de esa transacción, el sistema registra la reserva simple.
    41. El sistema crea una `OcupacionEspacio` asociada a la reserva.
    42. La ocupación creada bloquea la cancha para futuras reservas, eventos o actividades en el mismo horario.
    43. Si hubo seña o pago total, el sistema registra un pago asociado a la reserva.
    44. Si hubo pago, el sistema registra un movimiento de caja de tipo `INGRESO`.
    45. El sistema asigna el estado correspondiente:

        - `PENDIENTE`, si no hubo seña ni pago.
        - `SEÑADA`, si hubo seña y queda saldo pendiente.
        - `PAGADA`, si se pagó el importe total.

    46. El sistema guarda:

        - Fecha de creación.
        - Hora de creación.
        - Usuario que registró la reserva.

    47. El sistema muestra un resumen final.
    48. La reserva queda visible en la agenda de cancha.
    49. La reserva bloquea la disponibilidad de la cancha para ese día y horario.
---

## Regla transaccional obligatoria
La creación de una reserva debe ejecutarse de forma atómica. Esto significa que deben guardarse juntos:

- Reserva.
- OcupacionEspacio.
- Pago, si corresponde.
- MovimientoCaja, si corresponde.

Si falla cualquiera de esas operaciones, no debe guardarse ninguna. Por ejemplo:

Si se guarda la reserva, pero falla la creación de OcupacionEspacio, la cancha quedaría reservada en una tabla pero 
disponible en la agenda.
>Eso no debe ocurrir.
---

## Estados posibles de reserva

- `PENDIENTE`: la reserva fue registrada, pero no tiene seña ni pago.
- `SEÑADA`: la reserva tiene una seña registrada, pero todavía queda saldo pendiente.
- `PAGADA`: la reserva fue pagada en su totalidad.
- `CANCELADA`: la reserva fue cancelada.
- `REALIZADA`: la reserva ya ocurrió.
---

# Subflujo - Pago posterior del saldo de una reserva
Este subflujo se utiliza cuando una reserva quedó `PENDIENTE` o `SEÑADA` y luego la persona vuelve para pagar una seña,
un pago parcial o el saldo completo.

## Pasos

    1. El administrador busca la reserva.
    2. El sistema muestra:

        - Importe total.
        - Pagos ya registrados.
        - Saldo pendiente.
        - Estado actual.

    3. El administrador selecciona:
        - [ Registrar pago de reserva ]

    4. El administrador ingresa el monto recibido.
    5. El administrador selecciona método de pago.
    6. El sistema valida que el monto sea mayor a cero.
    7. El sistema valida que el monto no supere el saldo pendiente, salvo que se permita excedente configurable.
    8. Para la primera versión, no se permitirá excedente en pagos de reserva.
    9. El sistema muestra una previsualización.
    10. El administrador confirma.
    11. El sistema registra un pago asociado a la reserva.
    12. El sistema registra un movimiento de caja de tipo `INGRESO`.
    13. El sistema recalcula el saldo pendiente.
    14. Si el saldo pendiente queda en cero, la reserva queda `PAGADA`.
    15. Si todavía queda saldo pendiente, la reserva queda `SEÑADA`.

>Resultado:
El sistema permite completar el pago de una reserva sin crear una nueva reserva y sin duplicar ocupaciones de espacio.
---

# Subflujo - Cancelación de reserva
La cancelación debe conservar historial. Una reserva no debe eliminarse físicamente.

## Pasos

    1. El administrador busca la reserva.
    2. El sistema muestra los datos de la reserva, pagos asociados y ocupación vinculada.
    3. El administrador selecciona:
        - [ Cancelar reserva ]

    4. El sistema solicita motivo de cancelación.
    5. El sistema advierte si la reserva tiene pagos asociados.
    6. El administrador confirma la cancelación.
    7. El sistema cambia el estado de la reserva a `CANCELADA`.
    8. El sistema cambia el estado de la `OcupacionEspacio` asociada a `CANCELADA`.
    9. Al quedar cancelada la ocupación, la cancha se libera para nuevas reservas en ese horario.
    10. Los pagos asociados no se eliminan.
    11. El sistema registra usuario, fecha y hora de cancelación.

## Tratamiento del dinero ya pagado
Si la reserva tenía seña o pagos asociados, el sistema deberá permitir registrar qué ocurrió con ese dinero:

- `RETENIDO`: el complejo conserva la seña.
- `DEVUELTO_MANUALMENTE`: se devolvió dinero fuera del sistema o mediante un egreso registrado.
- `REPROGRAMADO`: el dinero queda asociado a una nueva reserva.
- `A_REVISAR`: queda pendiente de decisión administrativa.

Para la primera versión, la cancelación no deberá borrar pagos ni movimientos de caja. Si se devuelve dinero, deberá
registrarse mediante un flujo específico de devolución/anulación.

>No se recomienda usar `EGRESO` libre para devoluciones hasta definir un módulo de egresos o devoluciones controladas.

## Regla sobre reprogramación de reserva
Para la versión 1, se recomienda tratar una reprogramación como:

    Cancelación de la reserva original + creación de una nueva reserva.

Reglas:

- La reserva original conserva su historial.
- La ocupación original se cancela o libera.
- La nueva reserva crea una nueva `OcupacionEspacio`.
- Si existía dinero pagado, su tratamiento deberá quedar registrado como `REPROGRAMADO`.
- La asociación del pago original con la nueva reserva deberá resolverse mediante un flujo específico de reprogramación o ajuste administrativo.

>Esta decisión evita modificar una reserva histórica y perder trazabilidad.
---


# Subflujo - Marcar reserva como realizada
El estado `REALIZADA` representa que la reserva efectivamente ocurrió. Opciones posibles:

- Automáticamente, cuando la fecha y hora de fin ya pasaron.
- Manualmente, cuando el administrador confirma que la reserva ocurrió.
- Mediante un proceso programado que revise reservas vencidas.

Para la primera versión, se recomienda permitir la acción manual:
    - [ Marcar como realizada ]

Reglas:

- Solo se podrá marcar como realizada una reserva que no esté cancelada.
- Una reserva pendiente de pago puede marcarse como realizada, pero seguirá teniendo saldo pendiente.
- Marcar como realizada no genera movimiento de caja.
- La ocupación asociada puede pasar a estado `REALIZADA`.
---

## Pantalla - Resumen final de reserva

    Reserva registrada correctamente.
    
    Reserva N°: 15
    Estado: SEÑADA
    Fecha: 15/05/2026
    Horario: 18:00 a 19:00
    Responsable: Juan Pérez
    Teléfono: 11-5555-5555
    Tipo de reserva: Alquiler fútbol 5
    
    Importe total: $35.000
    Monto pagado: $10.000
    Saldo pendiente: $25.000
    Método de pago: Transferencia
    
    Caja:
        - Movimiento generado: INGRESO.
        - Concepto: ALQUILER_CANCHA.
        - Monto: $10.000.
    
    Espacio:
        - Cancha de fútbol 5 bloqueada de 18:00 a 19:00.
    
    Usuario que registró: Administrador


>El resumen de reserva no representa factura fiscal.
---

## Ejemplo 1 - alquiler simple de cancha

    Fecha: 15/05/2026
    Horario: 18:00 a 19:00
    Tipo: Alquiler fútbol 5
    Responsable: Juan Pérez
    Importe total: $35.000
    Seña: $10.000

    Resultado:
    
        - Se registra la reserva.
        - La cancha queda bloqueada de 18:00 a 19:00.
        - Se registra un pago por $10.000.
        - Se registra un movimiento de caja por $10.000.
        - La reserva queda en estado `SEÑADA`.
        - Queda saldo pendiente de $25.000.
---

## Ejemplo 2 - educación física escolar

    Fecha: 20/05/2026
    Horario: 10:00 a 12:00
    Tipo: Educación física escolar
    Responsable: Escuela San Martín
    Teléfono: 11-5555-5555
    Importe total: $50.000
    Seña: $0

    Resultado:
    
        - Se registra la reserva.
        - La cancha queda bloqueada de 10:00 a 12:00.
        - Se crea una ocupación de la cancha.
        - No se registra movimiento de caja porque no hubo pago.
        - La reserva queda en estado `PENDIENTE`.
---

## Ejemplo 3 - conflicto con escuela de fútbol

Existe:

    Ocupación: Escuela de fútbol
    Día: Martes
    Horario: 17:00 a 19:00
    Espacio: Cancha de fútbol 5

Se intenta registrar:

    Reserva simple de cancha
    Día: Martes
    Horario: 18:00 a 19:00

Resultado:
    
    El sistema no permite registrar la reserva.

Mensaje:
    La cancha no está disponible en ese horario.
    Ocupación existente: Escuela de fútbol
    Horario: 17:00 a 19:00.
---

## Ejemplo 4 - conflicto con cumpleaños deportivo

Existe:

    Evento: Cumpleaños deportivo mixto
    Horario: 17:00 a 19:00
    Espacio: Cancha de fútbol 5

Se intenta registrar:

    Reserva simple de cancha
    Horario: 18:00 a 19:00


Resultado:

    El sistema no permite registrar la reserva porque la cancha está ocupada por un evento.
---

## Ejemplo 5 - cancelación de reserva con seña

Reserva:

    Fecha: 15/05/2026
    Horario: 18:00 a 19:00
    Importe total: $35.000
    Seña pagada: $10.000
    Estado: SEÑADA

El administrador cancela la reserva. Resultado:

- La reserva queda en estado `CANCELADA`.
- La `OcupacionEspacio` queda en estado `CANCELADA`.
- La cancha se libera para ese horario.
- El pago de $10.000 no se elimina.
- El sistema solicita indicar si el dinero queda retenido, devuelto, reprogramado o a revisar.
---

## Decisiones importantes

- ¿El administrador tiene permiso para registrar reservas?
- ¿La reserva corresponde realmente a una reserva simple de cancha?
- ¿La fecha es válida?
- ¿La fecha está en el pasado?
- ¿La hora de inicio es anterior a la hora de fin?
- ¿La cancha está disponible?
- ¿Existe una reserva simple en ese horario?
- ¿Existe un evento que use la cancha en ese horario?
- ¿Existe escuela de fútbol en ese horario?
- ¿Existe educación física escolar en ese horario?
- ¿Existe bloqueo manual o mantenimiento en ese horario?
- ¿La persona es cliente registrado o eventual?
- ¿El importe total es mayor a cero?
- ¿La persona deja seña?
- ¿La seña es válida?
- ¿La seña cubre todo el importe?
- ¿Qué método de pago se utilizó?
- ¿La reserva queda pendiente, señada o pagada?
- ¿Se está pagando saldo de una reserva ya existente?
- ¿La reserva se cancela?
- Si se cancela con dinero pagado, ¿qué ocurre con ese dinero?
- ¿La reserva debe marcarse como realizada?
---

## Datos que intervienen

- Reserva.
- TipoReserva.
- Espacio.
- OcupacionEspacio.
- Cancha.
- Cliente, si corresponde.
- Persona eventual o responsable.
- Pago, si corresponde.
- PagoReserva o referencia `Pago -> Reserva`.
- MovimientoCaja, si corresponde.
- Método de pago.
- Usuario administrador.
- Auditoria.
- ResumenReserva o ComprobanteReserva.
---

## Nuevos conceptos detectados

- OcupacionEspacio:
Representa un bloqueo real sobre un espacio físico del complejo. Ejemplo:

    Cancha de fútbol 5
    15/05/2026
    18:00 a 19:00
    Tipo: RESERVA_CANCHA
    Origen: Reserva #15
    Estado: ACTIVA


- PagoReserva:
Representa la asociación entre un pago y una reserva. Puede implementarse como entidad propia o como referencia directa
desde `Pago` hacia `Reserva`, siempre que permita consultar todos los pagos realizados sobre una reserva.

- MotivoCancelacionReserva:
Permite registrar por qué una reserva fue cancelada y qué ocurrió con el dinero pagado.

---

## Reglas de negocio detectadas

- El Flujo 5 solo debe manejar reservas simples de cancha.
- Los cumpleaños deportivos no deben registrarse como reservas simples.
- Los cumpleaños deportivos deben registrarse desde el Flujo 6.
- El sistema debe verificar automáticamente la disponibilidad de la cancha antes de registrar una reserva simple.
- La disponibilidad debe considerar cualquier ocupación activa de la cancha.
- La escuela de fútbol bloquea la disponibilidad de la cancha durante sus horarios.
- Los cumpleaños deportivos bloquean la disponibilidad de la cancha durante sus horarios.
- La educación física escolar bloquea la disponibilidad de la cancha durante sus horarios.
- Una reserva simple no puede superponerse con otra reserva simple.
- Una reserva simple no puede superponerse con un cumpleaños deportivo.
- Una reserva simple no puede superponerse con horarios de escuela de fútbol.
- Una reserva simple no puede superponerse con educación física escolar.
- Una reserva simple no puede superponerse con bloqueos manuales o mantenimiento.
- La hora de inicio debe ser anterior a la hora de fin.
- El importe total debe ser mayor a cero.
- La seña no puede ser mayor al importe total.
- Una reserva puede estar asociada a un cliente registrado o a una persona eventual.
- Una reserva sin seña queda en estado `PENDIENTE`.
- Una reserva con seña y saldo pendiente queda en estado `SEÑADA`.
- Una reserva pagada completamente queda en estado `PAGADA`.
- Toda seña o pago de reserva debe generar movimiento de caja.
- Las reservas canceladas deben conservarse como historial.
- Al cancelar una reserva, se debe cancelar o liberar su `OcupacionEspacio`.
- Si la reserva tiene pagos asociados, no se elimina.
- Si se devuelve dinero, debe registrarse un egreso o un flujo de devolución/anulación.
- El pago posterior de saldo debe recalcular el saldo pendiente de la reserva.
- El estado `REALIZADA` puede asignarse manualmente o automáticamente al finalizar el horario.
- Las reservas deben guardar fecha, hora y usuario que las registró.
- En la primera versión, el saldo a favor del cliente no se aplicará a reservas de cancha.
- Al confirmar una reserva simple, el sistema debe crear una ocupación del espacio correspondiente.
- La creación de reserva, ocupación, pago y movimiento de caja debe ejecutarse en una única transacción.
- Toda creación de reserva deberá generar auditoría.
- Todo pago asociado a reserva deberá generar auditoría.
- Toda cancelación de reserva deberá generar auditoría con usuario, fecha, hora y motivo.
- Todo cambio de estado a `REALIZADA` deberá quedar auditado.
- En la versión 1, una reserva simple de cancha no podrá cruzar de un día a otro.
- Si el monto pagado inicialmente es igual al importe total, la reserva queda `PAGADA`.
- Si el monto pagado inicialmente es menor al importe total, la reserva queda `SEÑADA`.
- Si no hay pago inicial, la reserva queda `PENDIENTE`.
- En la versión 1, se recomienda tratar una reprogramación como cancelación de la reserva original más creación de una nueva reserva.
- En la versión 1, no se recomienda usar `EGRESO` libre para devoluciones hasta definir un flujo específico.
- El sistema deberá mostrar un resumen final de la reserva registrada.
- El resumen de reserva no representa factura fiscal.
---

# Flujo pendiente relacionado

Para que este flujo funcione completamente, se necesita un flujo específico para:
    Flujo 9 - Gestión de actividades, espacios y ocupaciones

Ese flujo permitirá registrar:

- Horarios fijos de escuela de fútbol.
- Educación física recurrente.
- Bloqueos por mantenimiento.
- Bloqueos manuales.
- Otros usos del espacio.
- Configuración de espacios disponibles del complejo.

Sin ese flujo, la reserva simple funciona para reservas manuales, pero no tendrá forma completa de conocer ocupaciones
recurrentes o bloqueos administrativos.
---

## Impacto en entidades
Confirmar o agregar estos atributos:

Reserva:
- id.
- fecha.
- horaInicio.
- horaFin.
- tipoReserva.
- responsable.
- telefono.
- cliente, si corresponde.
- importeTotal.
- totalPagado.
- saldoPendiente.
- estado.
- observacion.
- fechaCreacion.
- usuarioCreacion.
- fechaCancelacion.
- usuarioCancelacion.
- motivoCancelacion.
- tratamientoDineroCancelacion.
- fechaRealizacion.
- usuarioRealizacion.

OcupacionEspacio:
- id.
- espacio.
- fecha.
- horaInicio.
- horaFin.
- tipoOcupacion.
- estado.
- origenTipo.
- origenId.
- reserva.
- evento.
- actividad.
- observacion.
- fechaCreacion.
- usuarioCreacion.
---

## Resultado final
El sistema registra una reserva simple de cancha sin superposición de horarios. La cancha queda bloqueada para la fecha
y horario seleccionados mediante una `OcupacionEspacio`. Si hubo seña o pago total, el sistema registra el pago y el
movimiento de caja correspondiente. La reserva queda disponible para futuras consultas, pagos de saldo, cancelaciones,
agenda e informes.

Si la reserva se cancela, no se elimina: se conserva como historial, se libera la ocupación asociada y se registra qué
ocurrió con el dinero abonado. Los cumpleaños deportivos no se registran en este flujo; se registran desde el Flujo 6.

![Diagrama de flujo](./imagenes-de-los-flujos/flujo-5.png)

