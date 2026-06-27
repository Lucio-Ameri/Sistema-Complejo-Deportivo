# Reglas de negocio — Sistema de Gestión del Complejo Deportivo (Versión 1.0)

> **Objetivo del documento:** escribir, de forma explícita y verificable, las reglas que el sistema debe respetar. Cada 
> regla está pensada para poder convertirse luego en una validación de código, una restricción de base de datos o un 
> caso de prueba.

**Fuentes analizadas:** `alcance-version-1.md`, `entidades.md` y los 13 flujos de negocios (Flujos 1 a 13).

**Convenciones:**
- Las reglas se numeran con un prefijo por área (ej. `RN-CLI-03`) para poder referenciarlas desde código y pruebas.
- Las reglas marcadas con **(sugerida)** no estaban escritas de forma explícita en los documentos analizados, pero se 
consideran esenciales para la coherencia del sistema y se agregan aquí.

---

## RN-GEN — Reglas generales y transversales

- **RN-GEN-01** — Toda operación económica debe quedar registrada con fecha, hora, concepto, monto, método de pago y 
usuario responsable. (Regla central del sistema.)
- **RN-GEN-02** — Ningún registro con valor económico o histórico se elimina físicamente: las bajas se modelan como 
estados inactivos y las correcciones como anulaciones.
- **RN-GEN-03** — Toda anulación debe guardar motivo, usuario, fecha y hora.
- **RN-GEN-04** — Las operaciones que involucran varios registros relacionados (cliente + responsable + inscripción, 
reserva + ocupación + pago + caja, etc.) deben ejecutarse en una única transacción atómica: si falla una parte, no se 
guarda nada parcialmente.
- **RN-GEN-05** — Todos los importes monetarios se manejan con escala 2 y redondeo `HALF_UP`; nunca se usan `double` ni 
`float`.
- **RN-GEN-06** — Las operaciones aritméticas entre importes solo son válidas si comparten la misma moneda; en caso 
contrario debe rechazarse la operación.
- **RN-GEN-07** — Toda entidad mutable está protegida contra escrituras concurrentes (bloqueo optimista por versión).
- **RN-GEN-08** — El sistema debe mostrar mensajes claros y comprensibles para usuarios sin experiencia técnica, evitando 
exponer errores técnicos al usuario final.
- **RN-GEN-09** — Las operaciones importantes deben requerir confirmación explícita antes de ejecutarse.
- **RN-GEN-10** — Las personas eventuales (no registradas) pueden realizar compras, reservas, eventos y pagos, pero no 
pueden acumular saldo a favor ni generar deuda en V1.
- **RN-GEN-11 (sugerida)** — Toda fecha/hora del sistema debe interpretarse según la zona horaria configurada del negocio, 
para evitar inconsistencias en la caja diaria y los informes.

---

## RN-CLI — Clientes

- **RN-CLI-01** — Los clientes no se eliminan definitivamente; las bajas se registran cambiando el estado a inactivo.
- **RN-CLI-02** — Un cliente dado de baja queda en estado inactivo y conserva todo su historial.
- **RN-CLI-03** — Si se carga DNI, el sistema debe evitar duplicados de documento.
- **RN-CLI-04** — Se permite registrar clientes sin DNI cuando la configuración del sistema lo habilita 
(`permitirRegistroSinDNI`).
- **RN-CLI-05** — Si el cliente se registra sin DNI, el sistema debe advertir posibles duplicados por nombre, apellido, 
fecha de nacimiento y responsable (advertencia, no bloqueo).
- **RN-CLI-06** — La fecha de nacimiento no puede ser una fecha futura y debe permitir calcular una edad coherente.
- **RN-CLI-07** — Los datos mínimos obligatorios del cliente deben estar completos para poder guardarlo.
- **RN-CLI-08** — Un cliente puede tener uno o más responsables y estar inscripto en una o más actividades.
- **RN-CLI-09** — Todo cliente debe tener al menos un responsable principal. Si solo se carga un responsable, queda como 
principal automáticamente. (Aplica a clientes menores; si el cliente es mayor de edad, los responsables son opcionales.)
- **RN-CLI-10** — Todo cliente nuevo inicia con saldo a favor igual a cero.
- **RN-CLI-11** — El saldo a favor inicial en cero no debe generar un movimiento histórico de saldo (no hubo dinero ni 
ajuste).
- **RN-CLI-12** — La ficha del cliente debe mostrar su deuda total y su saldo a favor disponible.
- **RN-CLI-13** — Un cliente puede crearse y guardarse válidamente sin ninguna inscripción inicial; mientras no tenga 
inscripción activa, no participa de la generación de cuotas.
- **RN-CLI-14** — Debe poder buscarse un cliente por nombre, por apellido y por DNI (cuando exista).

---

## RN-RES — Responsables y vínculo cliente-responsable

- **RN-RES-01** — Un responsable es siempre mayor de edad (precondición validada al crearlo).
- **RN-RES-02** — Un responsable debe tener al menos nombre, apellido y teléfono.
- **RN-RES-03** — El DNI del responsable es opcional; si se informa, debe cumplir el formato normalizado.
- **RN-RES-04** — Si se carga email del responsable, debe tener formato válido.
- **RN-RES-05** — Un responsable puede estar asociado a uno o más clientes (relación N:M).
- **RN-RES-06** — Si el responsable ya existe (búsqueda por DNI, teléfono o nombre y apellido), debe poder asociarse al 
cliente sin crearlo de nuevo, evitando duplicados.
- **RN-RES-07** — El vínculo cliente-responsable debe registrar el parentesco/relación con el niño.
- **RN-RES-08** — Solo puede existir un responsable principal activo por cliente.
- **RN-RES-09** — Debe poder actualizarse la información de contacto del responsable sin perder el historial del cliente.

---

## RN-ACT — Actividades

- **RN-ACT-01** — Las actividades no se eliminan definitivamente; las que ya no se usan se marcan como inactivas.
- **RN-ACT-02** — Solo las actividades activas que permiten inscripción mensual se muestran al inscribir clientes; 
las actividades eventuales (confitería, alquiler de cancha, eventos particulares) no se ofrecen para inscripción.
- **RN-ACT-03** — Si una actividad no permite inscripción, tampoco puede generar cuota mensual (invariante: 
`permiteInscripcion = false` ⇒ `generaCuotaMensual = false`).
- **RN-ACT-04** — No todas las actividades generan cuota mensual; solo participan de la generación las que tienen 
`generaCuotaMensual = true`.
- **RN-ACT-05** — La escuela de fútbol está orientada a niños de 5 a 9 años.
- **RN-ACT-06** — El salón infantil está orientado a niños de 3 a 7 años, aunque admite eventos particulares compatibles 
con el espacio.
- **RN-ACT-07** — El precio base puede modificarse, pero los registros históricos (cuotas ya generadas) conservan el 
precio con el que fueron creados.
- **RN-ACT-08** — Si una actividad define edad mínima y máxima, la edad mínima no puede ser mayor que la máxima.
- **RN-ACT-09** — El precio base de una actividad no puede ser negativo.

---

## RN-ESP — Espacios físicos y ocupaciones

- **RN-ESP-01** — Solo los espacios físicos en estado activo pueden usarse en reservas, eventos y actividades.
- **RN-ESP-02** — Los espacios físicos no se borran; se marcan como inactivos.
- **RN-ESP-03** — No puede existir otro espacio activo con el mismo nombre.
- **RN-ESP-04** — Toda operación que use un espacio físico debe crear una ocupación de espacio que bloquee ese horario.
- **RN-ESP-05** — Solo las ocupaciones en estado activo bloquean la disponibilidad; las canceladas o anuladas no bloquean,
pero se conservan como historial.
- **RN-ESP-06** — Dos ocupaciones se consideran superpuestas cuando corresponden al mismo espacio, la misma fecha y sus 
rangos horarios se solapan.
- **RN-ESP-07** — No puede crearse una ocupación con hora de inicio posterior o igual a la hora de fin.
- **RN-ESP-08** — No puede crearse una ocupación si el espacio está inactivo.
- **RN-ESP-09** — No puede crearse una ocupación superpuesta con otra ocupación activa del mismo espacio.
- **RN-ESP-10** — La disponibilidad de un espacio debe verificarse contra TODAS las ocupaciones activas, sin importar su 
origen (reservas simples, eventos, escuela de fútbol, educación física escolar, actividades fijas, bloqueos manuales y 
mantenimiento).
- **RN-ESP-11** — Los bloqueos de mantenimiento impiden registrar reservas y eventos en ese horario.
- **RN-ESP-12** — Toda creación, modificación o cancelación de ocupación debe quedar auditada.
- **RN-ESP-13** — La validación de disponibilidad debe ocurrir ANTES de crear la ocupación (como precondición, no como 
reacción posterior).

---

## RN-INS — Inscripciones

- **RN-INS-01** — Una inscripción debe estar asociada a un cliente y a una actividad.
- **RN-INS-02** — No puede existir una inscripción activa duplicada para el mismo cliente y la misma actividad.
- **RN-INS-03** — Al inscribir, debe respetarse la restricción de edad de la actividad (edad mínima/máxima) cuando exista.
- **RN-INS-04** — El precio mensual de la inscripción debe ser mayor a cero.
- **RN-INS-05** — El día de vencimiento de la inscripción debe estar entre 1 y 28.
- **RN-INS-06** — La fecha de inicio (alta) es obligatoria y no puede estar vacía.
- **RN-INS-07** — Una fecha de inicio demasiado anterior o incoherente requiere confirmación explícita del administrador.
- **RN-INS-08** — Una inscripción nueva inicia en estado "activa" por defecto.
- **RN-INS-09** — El estado inicial de la inscripción no es editable en el alta común; solo se modifica desde un flujo 
especial o con permiso avanzado.
- **RN-INS-10** — Solo las inscripciones activas generan cuotas automáticas.
- **RN-INS-11** — Una inscripción finalizada no genera nuevas cuotas.
- **RN-INS-12** — Una inscripción suspendida no genera nuevas cuotas mientras se mantenga suspendida.

---

## RN-PRE — Precios mensuales de actividades

- **RN-PRE-01** — Para generar cuota de una actividad debe existir un precio vigente mayor a cero para ese período.
- **RN-PRE-02** — Un precio mensual ya usado por cuotas generadas no puede modificarse libremente; cambiar un precio 
futuro nunca altera cuotas de meses ya generados.
- **RN-PRE-03** — El precio definido para una actividad en un mes debe quedar guardado históricamente.
- **RN-PRE-04** — El importe de cada cuota se resuelve con esta prioridad: (1) precio mensual pactado en la inscripción, 
si existe; (2) precio mensual vigente de la actividad para el período; (3) precio por defecto de la actividad.

---

## RN-CUO — Cuotas

- **RN-CUO-01** — No puede registrarse una cuota con importe cero o negativo.
- **RN-CUO-02** — No puede existir una cuota duplicada para la misma combinación de cliente + actividad + período.
- **RN-CUO-03** — El saldo pendiente inicial de una cuota es igual a su importe.
- **RN-CUO-04** — Una cuota nueva se crea en estado pendiente.
- **RN-CUO-05** — Una cuota sin pagos queda pendiente; pagada parcialmente queda parcial; pagada totalmente queda pagada.
- **RN-CUO-06** — Una cuota queda pagada cuando su saldo pendiente llega a cero, sin importar si se cubrió con pago real, 
con saldo a favor o con ambos.
- **RN-CUO-07** — El estado de pago y el estado de vencimiento son dimensiones independientes: una cuota puede estar 
simultáneamente parcial (pago) y vencida (vencimiento).
- **RN-CUO-08** — Una cuota con saldo pendiente cuya fecha de vencimiento ya pasó debe figurar como vencida.
- **RN-CUO-09** — Cuando una cuota pasa a pagada o anulada, deja de tener saldo exigible (estado de vencimiento "sin deuda").
- **RN-CUO-10** — Una cuota anulada se conserva en el historial.
- **RN-CUO-11** — El sistema debe permitir saber exactamente qué mes o meses adeuda cada cliente.
- **RN-CUO-12** — La deuda del cliente se mide en meses (períodos mensuales), no en días exactos.

---

## RN-GCU — Generación mensual de cuotas

- **RN-GCU-01** — Las cuotas mensuales solo pueden generarse a partir del primer día hábil del mes (regla configurable); 
si el día 1 cae sábado o domingo, se habilita el lunes siguiente. En V1 no se consideran feriados.
- **RN-GCU-02** — No puede confirmarse dos veces una generación mensual para el mismo período.
- **RN-GCU-03** — El sistema debe impedir que dos usuarios generen cuotas del mismo período simultáneamente.
- **RN-GCU-04** — Solo se incluyen actividades activas con `generaCuotaMensual = true`; las actividades eventuales no 
aparecen en esta pantalla.
- **RN-GCU-05** — Cada actividad que genere cuota debe tener un precio mayor a cero para el período; no se permite 
generar si alguna actividad tiene precio cero, negativo o inválido.
- **RN-GCU-06** — Las cuotas se generan solo para inscripciones activas.
- **RN-GCU-07** — Si una inscripción inicia después del último día del período generado, no genera cuota (debe marcarse 
el motivo de omisión).
- **RN-GCU-08** — Las inscripciones iniciadas dentro del período deben mostrarse claramente en la previsualización.
- **RN-GCU-09** — Toda cuota omitida debe registrar el motivo de omisión (duplicado, sin precio vigente, inscripción 
inactiva, inicio posterior, actividad no mensual, error técnico).
- **RN-GCU-10** — La fecha de vencimiento de cada cuota se calcula con el día de vencimiento de la inscripción (1 a 28).
- **RN-GCU-11** — La generación debe ofrecer una previsualización antes de confirmar definitivamente.
- **RN-GCU-12** — Toda la generación de un período debe ejecutarse de forma atómica: si falla, no debe quedar una 
generación parcial inconsistente.
- **RN-GCU-13** — La generación de cuotas no genera ingresos de caja por sí misma (no entra dinero nuevo).
- **RN-GCU-14** — Al generar cuotas, el sistema debe verificar si el cliente posee saldo a favor y, si la opción está
activa, aplicarlo automáticamente a las cuotas del período.
- **RN-GCU-15** — La aplicación automática de saldo a favor es configurable y puede activarse/desactivarse por ejecución; 
si está desactivada, el saldo permanece disponible y la cuota queda como fue generada.
- **RN-GCU-16** — Durante la generación mensual, el saldo a favor solo se aplica a cuotas del período actual; no se 
aplica automáticamente a cuotas antiguas (eso corresponde a un flujo separado de cobro/compensación).
- **RN-GCU-17** — La aplicación de saldo a favor durante la generación no genera movimiento de caja nuevo.
- **RN-GCU-18** — Luego de generar las cuotas del período, el aviso de "generar cuotas" no debe volver a mostrarse para 
ese período.
- **RN-GCU-19** — El aviso de generación no debe mostrarse a usuarios sin permiso para generar cuotas.
- **RN-GCU-20** — El resumen final debe desglosar las cuotas generadas por estado de pago (pendientes, parciales, pagadas) 
y debe cumplirse: pendientes + parciales + pagadas = total generado.

---

## RN-PÁG. — Pagos y aplicación de pagos

- **RN-PÁG.-01** — No puede registrarse un pago con monto cero o negativo.
- **RN-PÁG.-02** — Todo pago debe tener fecha, hora, concepto, monto y método de pago.
- **RN-PÁG.-03** — Todo pago real debe impactar en la caja diaria (genera un movimiento de caja).
- **RN-PÁG.-04** — Los pagos no se eliminan; si fueron cargados por error, se anulan.
- **RN-PÁG.-05** — Un pago puede aplicarse a cuotas, reservas, eventos u otros conceptos cobrables; una operación puede 
pagarse en una o varias partes.
- **RN-PÁG.-06** — Debe poder seleccionarse y cobrarse más de una cuota en una sola operación de pago.
- **RN-PÁG.-07** — En V1, cada operación de pago usa un único método de pago (el diseño queda preparado para pagos mixtos 
futuros).
- **RN-PÁG.-08** — El monto recibido debe ser mayor a cero y debe seleccionarse un método de pago.
- **RN-PÁG.-09** — Cuando un pago parcial cubre varias cuotas, debe distribuirse en este orden: primero las cuotas vencidas, 
luego las pendientes más antiguas y por último las más recientes.
- **RN-PÁG.-10** — La distribución del pago entre cuotas debe poder previsualizarse antes de confirmar, mostrando cuánto 
se aplica a cada cuota.
- **RN-PÁG.-11** — Si el monto recibido supera el total seleccionado, el excedente solo puede registrarse como saldo a 
favor y requiere confirmación explícita.
- **RN-PÁG.-12** — Si el cliente tiene cuotas pendientes, pero el administrador intenta registrar un anticipo puro sin 
aplicar dinero a la deuda, el sistema debe advertirlo y requerir confirmación explícita (y, si se define así, permiso 
avanzado).
- **RN-PÁG.-13** — Si el cliente no tiene cuotas pendientes, el sistema debe permitir registrar un anticipo como saldo a 
favor.
- **RN-PÁG.-14** — El sistema debe registrar un único movimiento de caja por el monto total recibido, aunque el pago salde
varias cuotas.
- **RN-PÁG.-15** — La relación pago-cuota debe registrarse mediante una entidad intermedia de aplicación de pago (no basta
con un único campo de cuota en el pago).
- **RN-PÁG.-16** — La suma de los montos de los métodos de pago de un pago debe ser igual al total recibido.
- **RN-PÁG.-17** — La suma de los montos aplicados de un pago no puede superar el total recibido.
- **RN-PÁG.-18** — Todo pago debe generar un comprobante o resumen visible, que muestre saldo pendiente actualizado y saldo
a favor disponible luego del pago.
- **RN-PÁG.-19** — Todo pago registrado y toda generación de saldo a favor deben quedar auditados.

---

## RN-SAL — Saldo a favor del cliente

- **RN-SAL-01** — El saldo a favor solo puede pertenecer a un cliente registrado, nunca a una persona eventual.
- **RN-SAL-02** — El saldo a favor solo puede aplicarse a cuotas mensuales de actividades; no puede usarse para 
confitería/cafetería, alquileres de cancha, cumpleaños deportivos, salón infantil ni eventos particulares.
- **RN-SAL-03** — El saldo a favor nunca puede ser negativo.
- **RN-SAL-04** — El saldo a favor no puede eliminarse ni ajustarse sin dejar registro.
- **RN-SAL-05** — Toda generación, aplicación, anulación o ajuste de saldo a favor debe quedar registrada con tipo de 
movimiento, monto, saldo anterior/posterior, motivo, fecha, hora y usuario responsable.
- **RN-SAL-06** — Aplicar saldo a favor a una cuota NO genera movimiento de caja (el dinero ya fue registrado cuando 
ingresó).
- **RN-SAL-07** — Un pago superior al saldo pendiente genera saldo a favor por el excedente; el dinero recibido sí 
impacta en caja, pero la posterior aplicación de ese saldo no vuelve a impactar.
- **RN-SAL-08** — Un ajuste manual de saldo a favor (positivo o negativo) solo puede realizarlo el administrador y debe 
quedar justificado y auditado.
- **RN-SAL-09** — El monto de saldo aplicado a una cuota debe ser mayor a cero y no puede superar el saldo pendiente de 
la cuota al momento de la aplicación.
- **RN-SAL-10** — Cuando un cliente tiene varias cuotas del período, el saldo se aplica primero a las cuotas del período
actual y, dentro de él, en orden por actividad y luego por inscripción; el sobrante queda disponible para la siguiente 
cuota.
- **RN-SAL-11** — La ficha del cliente debe mostrar el saldo a favor disponible y permitir consultar el historial de 
movimientos de saldo.

---

## RN-PRO — Productos y stock

- **RN-PRO-01** — No puede crearse un producto sin nombre.
- **RN-PRO-02** — No puede crearse un producto con precio de venta cero o negativo.
- **RN-PRO-03** — No puede cargarse stock negativo (inicial ni resultante de un ajuste).
- **RN-PRO-04** — Al crear un producto, si ya existe otro producto activo con el mismo nombre, se muestra una advertencia 
NO bloqueante (el administrador puede continuar o cancelar).
- **RN-PRO-05** — El código de producto es opcional; si se informa, debe ser único.
- **RN-PRO-06** — Un producto con ventas asociadas no se elimina definitivamente; se marca como inactivo o sin stock.
- **RN-PRO-07** — Un producto inactivo no debe aparecer en la pantalla de venta rápida.
- **RN-PRO-08** — Cambiar el precio de un producto solo afecta a ventas futuras; nunca modifica ventas anteriores.
- **RN-PRO-09** — El stock nunca se modifica directamente: todo cambio se hace mediante un movimiento de stock y el 
servicio actualiza el stock actual de forma atómica.
- **RN-PRO-10** — Todo movimiento de stock se conserva como historial (registro append-only).
- **RN-PRO-11** — Todo ajuste de stock debe registrar motivo, fecha, usuario y cantidad.
- **RN-PRO-12** — Un producto tiene stock bajo cuando su stock actual es menor o igual a su stock mínimo, y el sistema 
debe permitir consultarlos.
- **RN-PRO-13** — Un producto con `controlaStock = false` (ej. café servido, agua caliente) se vende sin validar ni 
descontar stock.

---

## RN-VEN — Ventas de confitería / cafetería

- **RN-VEN-01** — Una venta debe tener al menos un producto; si el carrito queda vacío, no puede confirmarse.
- **RN-VEN-02** — No puede venderse un producto con cantidad cero o negativa.
- **RN-VEN-03** — No puede venderse un producto sin stock suficiente cuando el producto controla stock.
- **RN-VEN-04** — El total de la venta debe ser mayor a cero.
- **RN-VEN-05** — Toda venta debe cobrarse con un método de pago real; el saldo a favor no puede usarse en ventas (solo 
puede mostrarse como dato informativo).
- **RN-VEN-06** — Toda venta debe calcular automáticamente subtotal por producto y total general.
- **RN-VEN-07** — Toda venta guarda el precio unitario histórico vigente al momento de confirmar (no cambia si luego se 
modifica el precio del producto).
- **RN-VEN-08** — Si se agrega un producto que ya está en el carrito, se incrementa la cantidad existente en lugar de 
duplicar la línea.
- **RN-VEN-09** — El stock debe validarse en dos momentos: al agregar el producto y nuevamente al confirmar la venta.
- **RN-VEN-10** — Si al confirmar ya no hay stock suficiente, la venta no se guarda y se muestra el error correspondiente.
- **RN-VEN-11** — Toda venta descuenta stock de los productos que controlan stock y registra el movimiento de stock de 
salida.
- **RN-VEN-12** — Toda venta genera un movimiento de caja con concepto/área confitería-cafetería.
- **RN-VEN-13** — En V1, la venta se registra únicamente cuando se cobra; no se genera deuda por confitería/cafetería 
(no se fía).
- **RN-VEN-14** — Una venta nueva queda en estado registrada al confirmarse.
- **RN-VEN-15** — Las ventas no se eliminan físicamente; si se cargaron por error, se anulan.
- **RN-VEN-16** — Al anular una venta se debe: registrar fecha, hora, usuario y motivo; restaurar el stock de los 
productos que correspondan; registrar los movimientos de stock de entrada por anulación; y registrar un movimiento de caja 
de tipo anulación relacionado con la venta.
- **RN-VEN-17** — Una venta anulada debe seguir visible en auditoría e informes administrativos.

---

## RN-REV — Reservas simples de cancha

- **RN-REV-01** — El flujo de reservas simples no debe usarse para cumpleaños deportivos ni eventos infantiles 
(esos se registran como eventos).
- **RN-REV-02** — Antes de registrar una reserva, el sistema debe verificar la disponibilidad de la cancha contra 
cualquier ocupación activa (otras reservas, eventos, escuela de fútbol, educación física escolar, bloqueos y mantenimiento).
- **RN-REV-03** — Si existe una ocupación activa con horario superpuesto, no se permite registrar la reserva.
- **RN-REV-04** — La hora de inicio debe ser anterior a la hora de fin.
- **RN-REV-05** — En V1, una reserva no puede cruzar de un día a otro ni tener duración cero o extremadamente corta.
- **RN-REV-06** — El importe total de la reserva debe ser mayor a cero.
- **RN-REV-07** — La seña (o monto pagado inicial) debe ser mayor a cero cuando exista pago inicial, y no puede superar 
el importe total.
- **RN-REV-08** — En V1 no se permite excedente en pagos de reserva.
- **RN-REV-09** — El saldo pendiente de la reserva debe calcularse automáticamente; si llega a cero, la reserva puede 
quedar pagada.
- **RN-REV-10** — El saldo a favor del cliente no se aplica a reservas en V1.
- **RN-REV-11** — Toda seña o pago de reserva debe generar un movimiento de caja de tipo ingreso.
- **RN-REV-12** — Al confirmar la reserva, el sistema debe crear la ocupación del espacio correspondiente que bloquee la 
cancha en ese horario.
- **RN-REV-13** — La creación de reserva + ocupación + pago + movimiento de caja debe ejecutarse en una única transacción.
- **RN-REV-14** — Las reservas con pagos o movimientos asociados no se eliminan; la cancelación conserva el historial.
- **RN-REV-15** — Al cancelar una reserva se debe liberar (cancelar) su ocupación de espacio.
- **RN-REV-16** — Al cancelar una reserva con dinero abonado, debe registrarse el tratamiento del dinero (retenido, 
devolución parcial/total, reprogramado o a revisar); en V1 el saldo a favor no es un tratamiento válido para reservas.
- **RN-REV-17** — La cancelación no borra pagos ni movimientos de caja; cualquier devolución se registra mediante un flujo 
específico de devolución/anulación.
- **RN-REV-18** — Un pago posterior de saldo debe recalcular el saldo pendiente de la reserva; si el saldo vuelve a ser 
mayor a cero, la reserva no puede quedar pagada.
- **RN-REV-19** — Toda creación, pago, cancelación y cambio de estado a realizada de una reserva debe quedar auditado.

---

## RN-EVT — Cumpleaños y eventos infantiles

- **RN-EVT-01** — Cada evento debe tener un espacio físico asociado y generar una ocupación de ese espacio.
- **RN-EVT-02** — Antes de registrar un evento, el sistema debe verificar la disponibilidad del espacio contra cualquier 
ocupación activa.
- **RN-EVT-03** — No puede registrarse un evento en un espacio ocupado en el mismo horario; el sistema debe mostrar qué 
ocupación lo impide.
- **RN-EVT-04** — Para cumpleaños en salón infantil, la edad del cumpleañero de 3 a 7 años es una validación bloqueante: 
fuera de ese rango no se permite registrar el evento como cumpleaños de salón.
- **RN-EVT-05** — Para eventos particulares en salón infantil, una edad fuera del rango orientativo genera una advertencia 
NO bloqueante que, si el administrador continúa, debe quedar registrada en observaciones.
- **RN-EVT-06** — Para cumpleaños, nombre y edad del cumpleañero son obligatorios; para eventos particulares infantiles 
son opcionales.
- **RN-EVT-07** — La hora de inicio debe ser anterior a la hora de fin; en V1 un evento no puede cruzar de un día a otro.
- **RN-EVT-08** — El importe total del evento debe ser mayor a cero.
- **RN-EVT-09** — La seña (o pago inicial) debe ser mayor a cero cuando exista, y no puede superar el importe total.
- **RN-EVT-10** — La cantidad de invitados no puede ser negativa; cada servicio del evento debe tener cantidad mayor a 
cero y precio unitario no negativo.
- **RN-EVT-11** — El saldo a favor del cliente no se aplica a eventos en V1.
- **RN-EVT-12** — Toda seña o pago de evento debe generar un movimiento de caja de tipo ingreso.
- **RN-EVT-13** — La creación de evento + ocupación + pago + movimiento de caja debe ejecutarse en una única transacción.
- **RN-EVT-14** — El saldo pendiente del evento se calcula automáticamente; si llega a cero, el evento puede quedar pagado.
- **RN-EVT-15** — Los eventos con pagos asociados no se eliminan; la cancelación conserva el historial.
- **RN-EVT-16** — Al cancelar un evento se debe liberar su ocupación de espacio; los pagos no se borran y cualquier 
devolución se registra en un flujo específico.
- **RN-EVT-17** — Una reprogramación de evento se trata como cancelación del original más alta de uno nuevo, registrando 
el tratamiento del dinero como reprogramado cuando había pagos.
- **RN-EVT-18** — Un evento marcado como realizado que aún tenga saldo pendiente debe seguir figurando con saldo pendiente 
(realizado no implica pagado).
- **RN-EVT-19** — Toda creación, pago, cancelación y cambio de estado ha realizado de un evento debe quedar auditado.

---

## RN-CAJ — Caja diaria y movimientos de caja

- **RN-CAJ-01** — La caja diaria se basa exclusivamente en los movimientos de caja registrados; no recalcula ingresos por 
otra vía.
- **RN-CAJ-02** — Cada pago, venta, reserva o evento con dinero real debe generar un movimiento de caja.
- **RN-CAJ-03** — Los movimientos de caja no se eliminan; los anulados se marcan como anulados (registro append-only).
- **RN-CAJ-04** — Al anular una operación económica debe registrarse un movimiento de caja de tipo anulación que referencie 
al movimiento original.
- **RN-CAJ-05** — La aplicación de saldo a favor no se suma como ingreso de caja, porque ese dinero ya fue registrado 
cuando ingresó.
- **RN-CAJ-06** — Los movimientos anulados no deben sumarse como ingresos normales; deben mostrarse en una sección separada 
o restarse claramente del total neto.
- **RN-CAJ-07** — El sistema debe permitir consultar la caja por fecha y agrupar los ingresos por método de pago y por 
área del complejo.
- **RN-CAJ-08** — Cuando un mismo pago salda cuotas de actividades de distintas áreas, su movimiento de caja se clasifica 
como "otro" y la apertura por área se reconstruye desde las aplicaciones de pago.
- **RN-CAJ-09** — Cada movimiento de caja debe conservar una referencia a la operación que lo originó, para poder abrir su detalle.
- **RN-CAJ-10** — Este flujo solo consulta movimientos ya generados por otros flujos; no crea, modifica ni anula operaciones.
- **RN-CAJ-11** — En V1 no hay cierre formal de caja con arqueo; puede existir una caja diaria por fecha, creada 
automáticamente al registrar el primer movimiento del día. Solo puede haber una caja abierta a la vez.

---

## RN-CMP — Comprobantes

- **RN-CMP-01** — La numeración de comprobantes debe ser correlativa y obtenerse de forma atómica para evitar duplicados 
bajo concurrencia.
- **RN-CMP-02** — El número de comprobante es único por tipo de comprobante.
- **RN-CMP-03** — Un comprobante anulado no puede reactivarse.

---

## RN-ANU — Anulaciones y correcciones

- **RN-ANU-01** — Las operaciones económicas (pagos, ventas, reservas, eventos) no se eliminan; se anulan conservando el 
historial.
- **RN-ANU-02** — Toda anulación requiere motivo, usuario, fecha y hora.
- **RN-ANU-03** — No puede anularse dos veces la misma operación.
- **RN-ANU-04** — Anular un pago de cuota debe recalcular la deuda; una cuota que estaba pagada puede volver a 
pendiente/parcial y, si su vencimiento ya pasó, volver a vencida.
- **RN-ANU-05** — Anular una venta debe restaurar el stock.
- **RN-ANU-06** — Anular un pago de reserva o de evento debe recalcular su saldo pendiente; si el saldo vuelve a ser mayor 
a cero, la operación no puede quedar pagada.
- **RN-ANU-07** — Si el pago anulado había generado saldo a favor, debe registrarse un movimiento de saldo de anulación 
que debite ese saldo.
- **RN-ANU-08** — Toda anulación económica debe reflejarse en la caja diaria como movimiento de anulación cuando hubo 
dinero real involucrado.
- **RN-ANU-09** — Las operaciones anuladas no deben sumarse como ingresos normales pero deben seguir visibles en auditoría 
e informes.
- **RN-ANU-10** — La auditoría debe conservar quién creó y quién anuló cada operación.
- **RN-ANU-11** — En principio solo el administrador puede anular operaciones críticas; el encargado solo puede hacerlo 
si el administrador le otorga el permiso.
- **RN-ANU-12** — Si se necesita devolver dinero, debe registrarse mediante un flujo específico de devolución/egreso, no 
borrando pagos.

---

## RN-INF — Informes y gráficos

- **RN-INF-01** — Los informes solo consultan información generada por otros flujos; no crean, modifican, anulan ni 
eliminan operaciones.
- **RN-INF-02** — Cada informe debe respetar los permisos del usuario; si no tiene permiso suficiente, no se genera.
- **RN-INF-03** — Los informes de ingresos deben excluir los movimientos anulados de los totales normales; si se muestran, 
deben figurar separados o restarse claramente del total neto.
- **RN-INF-04** — Los gráficos deben usar exactamente la misma base de datos, filtros y exclusiones que las tablas.
- **RN-INF-05** — La fecha "desde" no puede ser posterior a la fecha "hasta".
- **RN-INF-06** — En informes grandes, el rango de fechas es obligatorio; si la consulta es demasiado amplia, el sistema 
debe pedir filtros más específicos.
- **RN-INF-07** — Los informes deben paginar los resultados cuando hay muchos registros; los gráficos trabajan con datos 
resumidos.
- **RN-INF-08** — Los informes de ingresos deben permitir abrir el origen real del movimiento que generó cada registro.
- **RN-INF-09** — El saldo a favor aplicado se muestra como información separada y no se suma al ingreso del día.

---

## RN-USR — Usuarios, roles, seguridad y auditoría

- **RN-USR-01** — Cada usuario debe ingresar con usuario y contraseña; un usuario inactivo no puede ingresar.
- **RN-USR-02** — Las contraseñas nunca se guardan ni procesan en texto plano (siempre hasheadas).
- **RN-USR-03** — El nombre de usuario debe ser único en el sistema.
- **RN-USR-04** — No puede desactivarse (ni cambiarse de rol) al último administrador activo.
- **RN-USR-05** — No deben compartirse usuarios entre varias personas.
- **RN-USR-06** — No todos los usuarios pueden acceder a todas las funciones; el acceso depende del rol y sus permisos.
- **RN-USR-07** — Los datos de clientes menores de edad solo deben visualizarse por usuarios autorizados.
- **RN-USR-08** — Toda operación crítica debe quedar auditada con usuario ejecutor, fecha/hora, acción, entidad afectada
y, cuando corresponda, valores anterior/nuevo y motivo.
- **RN-USR-09** — Los registros de auditoría son de solo escritura (append-only): nunca se modifican una vez creados.
- **RN-USR-10** — La consulta de auditoría debe permitir filtrar por rango de fechas, usuario y tipo de operación.
- **RN-USR-11** — El sistema no debe mostrar información sensible innecesaria, y si se publica en red debe usar conexión 
segura sin exponer la base de datos directamente a internet.
- **RN-USR-12 (sugerida)** — Toda contraseña debe cumplir condiciones mínimas de complejidad/longitud al crearse o
modificarse.

---

## RN-BCK — Respaldo y restauración

- **RN-BCK-01** — Debe realizarse un backup periódico de la base de datos.
- **RN-BCK-02** — Antes de actualizar el sistema debe realizarse una copia de seguridad.
- **RN-BCK-03** — La información no debe depender de una sola computadora; debe guardarse al menos una copia externa.
- **RN-BCK-04** — Debe probarse periódicamente que los backups puedan restaurarse correctamente.
- **RN-BCK-05** — El procedimiento de backup debe estar documentado.
- **RN-BCK-06** — Cada backup manual debe quedar auditado y registrar si fue exitoso, fallido o incompleto, y dónde 
quedó almacenado.
- **RN-BCK-07** — Antes de restaurar, debe validarse la integridad del backup (checksum).
- **RN-BCK-08** — La restauración requiere confirmación fuerte y solo puede realizarla un administrador.
- **RN-BCK-09** — Toda restauración debe quedar registrada con quién la realizó, cuándo, desde qué backup y con qué 
resultado (incluso si falla).
- **RN-BCK-10** — Si un backup automático falla, el sistema debe avisar al administrador en el próximo inicio.

---

## RN-CFG — Configuración del sistema

- **RN-CFG-01** — La configuración general del sistema es un singleton lógico: solo existe un registro activo.
- **RN-CFG-02** — Las reglas configurables (día de vencimiento por defecto, regla de primer día hábil, recargo por mora, 
permitir registro sin DNI, permitir saldo a favor, aplicación automática de saldo a favor) deben poder cambiarse sin 
modificar el código.
- **RN-CFG-03** — El día de vencimiento configurable debe estar entre 1 y 28, tanto a nivel sistema como por inscripción.
- **RN-CFG-04 (sugerida)** — Los cambios en la configuración crítica del sistema deben quedar auditados (quién, cuándo y 
qué valor anterior/nuevo).

---

## Resumen

Este documento consolida **más de 180 reglas de negocio** extraídas de los documentos de alcance, entidades y flujos, 
organizadas en 21 áreas funcionales. Las reglas marcadas como **(sugerida)** son agregados propuestos para reforzar la 
coherencia del sistema y pueden discutirse o ajustarse según las necesidades reales del complejo.

> **Regla central que rige todo el sistema:**
> Toda operación económica debe quedar registrada con fecha, hora, concepto, monto, método de pago y usuario responsable, 
> y ninguna operación con valor económico o histórico se elimina: se anula o se inactiva, conservando siempre la trazabilidad.
