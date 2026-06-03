# Alcance de la Versión 1.0 - Sistema de Gestión para el Complejo Deportivo


---

# 1. Objetivo general del sistema

El objetivo del sistema es reemplazar progresivamente el uso de papel y lapicera en la administración diaria del complejo deportivo, permitiendo registrar, consultar y controlar de forma clara y segura la información relacionada con clientes, responsables, actividades, cuotas, pagos, ventas, reservas, eventos, caja diaria, informes y gráficos.

La finalidad principal es que el complejo pueda responder rápidamente preguntas como:

- ¿Qué clientes están activos?
- ¿Qué actividad realiza cada cliente?
- ¿Quién pagó?
- ¿Quién debe?
- ¿Qué mes o meses debe cada cliente?
- ¿Cuándo vence cada cuota?
- ¿Qué método de pago se utilizó?
- ¿Qué productos se vendieron?
- ¿Qué reservas hay para un día determinado?
- ¿Qué eventos infantiles están pendientes?
- ¿Cuánto ingresó hoy?
- ¿Cuánto ingresó por cada área del complejo?
- ¿Cuánto ingresó por efectivo, débito, crédito, transferencia o Mercado Pago?
- ¿Qué productos tienen stock bajo?
- ¿Qué usuario registró cada operación?

El sistema deberá ser simple de usar, claro visualmente y comprensible para personas con poca o ninguna experiencia previa utilizando sistemas informáticos de gestión.

---

# 2. Descripción general del negocio

El complejo deportivo cuenta con distintas áreas físicas y comerciales, cada una con sus propias actividades e ingresos.

## 2.1 Cancha de fútbol 5

En la cancha de fútbol 5 se realizan las siguientes actividades:

- Alquiler de cancha para fútbol 5.
- Cumpleaños deportivos para varones.
- Cumpleaños deportivos mixtos.
- Escuela de fútbol para niños de 5 a 9 años.
- Alquiler de cancha para escuelas que realizan actividades de educación física.

## 2.2 Piso intermedio

En el piso intermedio se realizan clases de Taekwondo.

Esta actividad puede generar ingresos mediante cuotas mensuales de alumnos inscriptos.

## 2.3 Segundo piso

En el segundo piso hay un salón de fiestas infantiles con pelotero.

Este espacio está orientado principalmente a:

- Cumpleaños infantiles.
- Niños de entre 3 y 7 años.
- Bautismos infantiles.
- Fiestas de un año.
- Otros eventos particulares compatibles con el espacio.

## 2.4 Confitería / cafetería

El complejo también realiza ventas de productos de consumo, por ejemplo:

- Bebidas.
- Golosinas.
- Snacks.
- Cafetería.
- Otros productos.

Estas ventas podrán realizarse tanto a clientes registrados como a personas eventuales no registradas en el sistema.

---

# 3. Espacios físicos del complejo

El sistema deberá reconocer los espacios físicos principales del complejo, ya que serán necesarios para organizar actividades, reservas, eventos e informes.

Los espacios iniciales serán:

- Cancha de fútbol 5.
- Piso intermedio / sala de Taekwondo.
- Salón infantil del segundo piso.
- Confitería / cafetería.

Cada espacio podrá asociarse a una o varias actividades.

Ejemplos:

| Espacio físico | Usos posibles |
|---|---|
| Cancha de fútbol 5 | Alquiler de cancha, escuela de fútbol, cumpleaños deportivos, educación física escolar |
| Piso intermedio | Taekwondo |
| Salón infantil | Cumpleaños infantiles, eventos particulares |
| Confitería / cafetería | Ventas de productos |

---

# 4. Problemas actuales que el sistema debe resolver

Actualmente el complejo se administra de forma manual mediante papel y lapicera. Esto genera los siguientes problemas:

- Dificultad para saber qué cliente pagó.
- Dificultad para saber qué cliente debe.
- Dificultad para conocer exactamente qué mes debe cada cliente.
- Riesgo de pérdida de información.
- Posibilidad de errores en cuentas manuales.
- Falta de historial confiable de pagos.
- Falta de historial de compras.
- Dificultad para calcular ingresos diarios.
- Dificultad para separar ingresos por área.
- Dificultad para controlar ventas de confitería.
- Dificultad para controlar stock.
- Dificultad para detectar productos con stock bajo.
- Dificultad para controlar reservas de cancha.
- Dificultad para evitar reservas superpuestas.
- Dificultad para controlar señas y saldos de cumpleaños.
- Falta de reportes automáticos.
- Falta de gráficos e informes visuales.
- Falta de control sobre quién registró cada operación.
- Riesgo de perder información histórica al corregir errores.

---

# 5. Alcance de la Versión 1.0

La versión 1.0 incluirá los módulos necesarios para administrar el funcionamiento básico e integral del complejo.

Los módulos incluidos serán:

1. Clientes.
2. Responsables.
3. Actividades.
4. Inscripciones.
5. Cuotas.
6. Pagos.
7. Confitería / productos.
8. Ventas.
9. Reservas de cancha.
10. Cumpleaños y eventos.
11. Caja diaria.
12. Informes.
13. Gráficos.
14. Usuarios y roles básicos.
15. Seguridad y privacidad.
16. Auditoría.
17. Respaldo de información.

---

# 6. Módulos incluidos en la Versión 1.0

---

## 6.1 Clientes

El sistema permitirá administrar los clientes del complejo.

En esta versión, los clientes principales serán los niños que realizan actividades deportivas dentro del complejo.

El sistema permitirá:

- Registrar clientes.
- Editar datos de clientes.
- Dar de baja clientes sin eliminarlos definitivamente.
- Consultar clientes activos.
- Consultar clientes inactivos.
- Buscar clientes por nombre.
- Buscar clientes por apellido.
- Buscar clientes por DNI, si fue cargado.
- Ver la situación económica de cada cliente.
- Ver actividades asociadas a cada cliente.
- Ver historial de pagos.
- Ver historial de compras.
- Ver cuotas pendientes, parciales, pagadas y vencidas.

Datos mínimos de un cliente:

- Nombre.
- Apellido.
- DNI, opcional en la versión 1.0.
- Fecha de nacimiento.
- Teléfono de contacto.
- Observaciones.
- Estado.
- Fecha de alta.
- Fecha de baja, si corresponde.

Reglas:

- No se deberán borrar clientes definitivamente.
- Los clientes dados de baja deberán quedar en estado inactivo.
- Si se carga DNI, el sistema deberá evitar duplicados.
- El sistema deberá permitir clientes sin DNI en caso de que el dato no esté disponible al momento del alta.
- Un cliente podrá tener uno o más responsables asociados.
- Un cliente podrá estar inscripto en una o más actividades.

---

## 6.2 Responsables

El sistema permitirá registrar los datos de adultos responsables asociados a los clientes.

Datos mínimos de un responsable:

- Nombre.
- Apellido.
- DNI.
- Teléfono.
- Email.
- Domicilio.
- Parentesco o relación con el niño.

Reglas:

- Un cliente podrá tener uno o más responsables.
- Un responsable podrá estar asociado a uno o más clientes.
- El teléfono del responsable será un dato importante para contacto operativo.
- El sistema deberá permitir actualizar datos de contacto sin perder historial del cliente.

---

## 6.3 Actividades

El sistema permitirá administrar las actividades del complejo.

Actividades iniciales:

- Escuela de fútbol.
- Taekwondo.
- Alquiler de cancha fútbol 5.
- Educación física escolar.
- Cumpleaños deportivo para varones.
- Cumpleaños deportivo mixto.
- Salón infantil.
- Confitería / cafetería.
- Evento particular.

Cada actividad deberá registrar:

- Nombre.
- Tipo.
- Espacio físico asociado, si corresponde.
- Edad mínima, si corresponde.
- Edad máxima, si corresponde.
- Precio base.
- Estado activa o inactiva.

Reglas:

- Las actividades no deberán eliminarse definitivamente.
- Las actividades que ya no se usen deberán marcarse como inactivas.
- La escuela de fútbol estará orientada a niños de 5 a 9 años.
- El salón infantil estará orientado a niños de entre 3 y 7 años, aunque podrá aceptar eventos particulares compatibles con el espacio.
- El precio base podrá modificarse, pero los registros históricos deberán conservar el precio con el que fueron generados.

---

## 6.4 Inscripciones

El sistema permitirá inscribir clientes a actividades.

Una inscripción deberá registrar:

- Cliente.
- Actividad.
- Fecha de inicio.
- Fecha de finalización, si corresponde.
- Precio mensual.
- Día de vencimiento.
- Estado de la inscripción.

Estados posibles:

- Activa.
- Suspendida.
- Finalizada.

Reglas:

- Solo se podrán generar cuotas automáticas para inscripciones activas.
- El día de vencimiento deberá ser válido.
- El precio mensual deberá ser mayor a cero.
- Una inscripción finalizada no deberá generar nuevas cuotas.
- Una inscripción suspendida no debería generar nuevas cuotas mientras se mantenga suspendida.
- El sistema deberá validar edad mínima y máxima cuando la actividad tenga restricciones de edad.

---

## 6.5 Cuotas

El sistema permitirá generar, consultar y controlar cuotas mensuales de clientes inscriptos en actividades.

Cada cuota deberá registrar:

- Cliente.
- Actividad.
- Mes correspondiente.
- Fecha de vencimiento.
- Importe.
- Saldo pendiente.
- Estado.

Estados posibles:

- Pendiente.
- Parcial.
- Pagada.
- Vencida.
- Anulada.

Reglas:

- No se podrá registrar una cuota con importe cero o negativo.
- No se deberá crear una cuota duplicada para el mismo cliente, actividad y mes.
- El saldo pendiente inicial de una cuota deberá ser igual al importe.
- Una cuota sin pagos deberá quedar pendiente.
- Una cuota pagada parcialmente deberá quedar parcial.
- Una cuota pagada completamente deberá quedar pagada.
- Una cuota vencida con saldo pendiente deberá quedar vencida.
- Una cuota anulada deberá conservarse en el historial.
- El sistema deberá permitir saber exactamente qué mes o meses debe cada cliente.

---

## 6.6 Pagos

El sistema permitirá registrar pagos realizados por clientes registrados o personas eventuales.

Cada pago deberá registrar:

- Cliente, si corresponde.
- Persona eventual, si corresponde.
- Monto.
- Método de pago.
- Concepto.
- Fecha y hora.
- Observaciones.
- Usuario que registró el pago.
- Estado del pago.

Métodos de pago iniciales:

- Efectivo.
- Débito.
- Crédito.
- Transferencia.
- Mercado Pago.
- Otro.

Conceptos iniciales:

- Cuota fútbol.
- Cuota Taekwondo.
- Confitería / cafetería.
- Alquiler de cancha.
- Cumpleaños deportivo.
- Salón infantil.
- Educación física escolar.
- Evento particular.
- Otro.

Estados posibles:

- Registrado.
- Anulado.

Reglas:

- No se podrá registrar un pago con monto cero o negativo.
- Todo pago deberá tener fecha, hora, concepto, monto y método de pago.
- Todo pago deberá impactar en caja diaria.
- Los pagos no deberán eliminarse.
- Si un pago fue cargado por error, deberá anularse.
- Toda anulación deberá guardar fecha, usuario y motivo.
- Si se anula un pago asociado a una cuota, la deuda deberá recalcularse.
- Los pagos podrán estar asociados a cuotas, ventas, reservas, eventos u otros conceptos.
- Una operación podrá pagarse en una o varias partes.

---

## 6.6.1 Saldo a favor de clientes para futuras cuotas

El sistema deberá permitir que un cliente tenga saldo a favor cuando realiza un pago superior al saldo pendiente de una o varias cuotas.

Este saldo a favor tendrá una restricción obligatoria:

> El saldo a favor del cliente solo podrá utilizarse para pagar futuras cuotas del mismo cliente.

No podrá utilizarse para otros conceptos del complejo.

### Alcance del saldo a favor

El saldo a favor podrá generarse en situaciones como:

- El cliente paga un monto mayor al saldo pendiente de una cuota.
- El cliente adelanta dinero para una cuota futura.
- El administrador registra un ajuste manual justificado a favor del cliente.

Ejemplo:

```text
Cuota de mayo: $25.000
Pago recibido: $30.000

Resultado:
- La cuota de mayo queda pagada.
- El cliente queda con $5.000 de saldo a favor.
```

Ese saldo a favor podrá aplicarse luego, por ejemplo, a la cuota de junio.

```text
Cuota de junio: $25.000
Saldo a favor disponible: $5.000

Resultado:
- Se aplican $5.000 de saldo a favor.
- El cliente debe pagar $20.000 restantes.
- El saldo a favor queda en $0.
```

### Usos permitidos

El saldo a favor solo podrá aplicarse a:

- Cuotas de escuela de fútbol.
- Cuotas de Taekwondo.
- Otras cuotas mensuales de actividades que existan en el sistema.

### Usos no permitidos

El saldo a favor no podrá utilizarse para:

- Compras de confitería o cafetería.
- Alquileres de cancha.
- Cumpleaños deportivos.
- Salón infantil.
- Eventos particulares.
- Educación física escolar, salvo que se modele como cuota mensual de un cliente o institución.
- Otros conceptos que no sean cuotas.

### Reglas de negocio del saldo a favor

- Un cliente podrá tener saldo a favor acumulado.
- El saldo a favor deberá estar asociado a un cliente registrado.
- El saldo a favor no podrá pertenecer a una persona eventual.
- El saldo a favor no podrá ser negativo.
- El saldo a favor no deberá eliminarse manualmente sin dejar registro.
- Toda generación de saldo a favor deberá quedar registrada.
- Toda aplicación de saldo a favor a una cuota deberá quedar registrada.
- Toda anulación o ajuste de saldo a favor deberá guardar motivo, fecha, hora y usuario responsable.
- El sistema deberá mostrar el saldo a favor en la ficha del cliente.
- El sistema deberá permitir consultar el historial de movimientos de saldo a favor.
- El saldo a favor no reemplaza a los movimientos de caja: si el cliente entrega dinero, ese ingreso debe impactar en caja diaria.
- Cuando el saldo a favor se aplica a una cuota futura, no debe duplicarse el ingreso en caja, porque el dinero ya fue registrado cuando se recibió.

### Impacto en cuotas

Cuando se registre un pago de cuota, el sistema deberá contemplar estos casos:

#### Pago menor al saldo pendiente

```text
Cuota: $25.000
Pago: $10.000

Resultado:
- Saldo pendiente: $15.000
- Estado de cuota: Parcial
- Saldo a favor generado: $0
```

#### Pago igual al saldo pendiente

```text
Cuota: $25.000
Pago: $25.000

Resultado:
- Saldo pendiente: $0
- Estado de cuota: Pagada
- Saldo a favor generado: $0
```

#### Pago mayor al saldo pendiente

```text
Cuota: $25.000
Pago: $30.000

Resultado:
- Saldo pendiente: $0
- Estado de cuota: Pagada
- Saldo a favor generado: $5.000
```

### Aplicación de saldo a favor

El sistema deberá permitir aplicar saldo a favor a cuotas futuras.

Ejemplo:

```text
Cliente: Mateo Gómez
Saldo a favor disponible: $5.000
Cuota de junio: $25.000

Aplicación:
- Se aplican $5.000 de saldo a favor.
- La cuota queda con saldo pendiente de $20.000.
- El saldo a favor del cliente queda en $0.
```

Esta aplicación deberá quedar registrada como un movimiento de saldo, pero no como un nuevo ingreso de caja.

### Información mínima que debe registrar cada movimiento de saldo

Cada movimiento de saldo a favor deberá guardar:

- Cliente.
- Fecha y hora.
- Tipo de movimiento.
- Monto.
- Motivo.
- Usuario responsable.
- Cuota que originó el saldo, si corresponde.
- Cuota donde se aplicó el saldo, si corresponde.
- Observaciones.

Tipos iniciales de movimiento de saldo:

- Generación de saldo a favor.
- Aplicación de saldo a cuota.
- Ajuste manual positivo.
- Ajuste manual negativo.
- Anulación.

### Informes relacionados con saldo a favor

El sistema deberá permitir consultar:

- Clientes con saldo a favor.
- Total de saldo a favor acumulado.
- Historial de movimientos de saldo por cliente.
- Cuotas donde se aplicó saldo a favor.
- Saldos a favor generados por pagos excedentes.

### Consideración técnica inicial

Para implementar esta funcionalidad, se recomienda agregar una entidad específica para registrar el historial de saldo a favor, por ejemplo:

```text
MovimientoSaldoCliente
```

Además, el cliente podrá tener un campo resumen:

```text
saldoAFavorCuotas
```

Ese campo servirá para consultar rápidamente el saldo disponible, pero el historial real deberá mantenerse en los movimientos de saldo.

---

## 6.7 Confitería / productos

El sistema permitirá registrar y administrar productos de confitería / cafetería.

Cada producto deberá registrar:

- Nombre.
- Categoría.
- Precio de venta.
- Stock actual.
- Stock mínimo.
- Estado activo o inactivo.

Reglas:

- No se podrá vender un producto sin stock suficiente.
- Toda venta deberá descontar stock automáticamente.
- El sistema deberá permitir consultar productos con stock bajo.
- Un producto tendrá stock bajo cuando su stock actual sea menor o igual a su stock mínimo.
- Los productos no deberán eliminarse definitivamente si tienen ventas asociadas.
- Si un producto ya no se vende, deberá marcarse como inactivo.
- El precio histórico de venta deberá conservarse dentro del detalle de cada venta.

---

## 6.8 Ventas

El sistema permitirá registrar ventas de productos de confitería / cafetería.

Cada venta deberá registrar:

- Cliente, si corresponde.
- Comprador eventual, si corresponde.
- Fecha y hora.
- Productos vendidos.
- Cantidades.
- Precio unitario.
- Subtotal por producto.
- Total de la venta.
- Método de pago.
- Usuario que registró la venta.
- Estado de la venta.

Estados posibles:

- Registrada.
- Anulada.

Reglas:

- El sistema deberá permitir ventas a clientes registrados.
- El sistema deberá permitir ventas a personas no registradas.
- Una venta deberá tener al menos un producto.
- Una venta deberá calcular automáticamente subtotales y total.
- Una venta deberá descontar stock.
- Toda venta deberá generar un movimiento de caja.
- Las ventas no deberán eliminarse.
- Si una venta fue cargada por error, deberá anularse.
- Si se anula una venta, el stock deberá restaurarse.
- Toda anulación deberá guardar fecha, usuario y motivo.

---

## 6.9 Reservas de cancha

El sistema permitirá registrar reservas de la cancha de fútbol 5.

Cada reserva deberá registrar:

- Fecha.
- Hora de inicio.
- Hora de fin.
- Tipo de reserva.
- Espacio físico.
- Nombre del responsable.
- Teléfono.
- Importe total.
- Total pagado.
- Saldo pendiente.
- Estado.
- Observaciones.

Tipos de reserva iniciales:

- Alquiler fútbol 5.
- Educación física escolar.
- Cumpleaños deportivo.

Estados posibles:

- Pendiente.
- Señada.
- Pagada.
- Cancelada.
- Realizada.

Reglas:

- No se podrá reservar la cancha en un horario ya ocupado.
- El sistema deberá validar superposición por fecha, horario y espacio físico.
- Una reserva podrá pagarse en una o varias partes.
- Una reserva podrá tener seña inicial.
- El saldo pendiente deberá calcularse automáticamente.
- Si el saldo pendiente llega a cero, la reserva podrá quedar pagada.
- Una reserva cancelada deberá conservar su historial.
- No se deberán eliminar reservas con movimientos económicos asociados.
- Todo pago de reserva deberá impactar en caja diaria.

---

## 6.10 Cumpleaños y eventos

El sistema permitirá registrar cumpleaños y eventos infantiles.

Tipos de evento iniciales:

- Cumpleaños deportivo para varones.
- Cumpleaños deportivo mixto.
- Salón infantil.
- Evento particular.

Cada evento deberá registrar:

- Tipo de evento.
- Fecha.
- Hora de inicio.
- Hora de fin.
- Espacio físico.
- Nombre del cumpleañero, si corresponde.
- Edad del cumpleañero, si corresponde.
- Nombre del responsable.
- Teléfono del responsable.
- Importe total.
- Total pagado.
- Saldo pendiente.
- Estado.
- Observaciones.

Estados posibles:

- Pendiente.
- Señado.
- Pagado.
- Cancelado.
- Realizado.

Reglas:

- El salón infantil estará orientado a niños de entre 3 y 7 años.
- El salón infantil también podrá utilizarse para eventos particulares compatibles.
- Un cumpleaños deportivo podrá ser para varones o mixto.
- No se deberán permitir eventos superpuestos en el mismo espacio, fecha y horario.
- Un evento podrá pagarse en una o varias partes.
- Un evento podrá tener seña inicial.
- El saldo pendiente deberá calcularse automáticamente.
- Si el saldo pendiente llega a cero, el evento podrá quedar pagado.
- Un evento cancelado deberá conservar su historial.
- No se deberán eliminar eventos con movimientos económicos asociados.
- Todo pago de evento deberá impactar en caja diaria.

---

## 6.11 Relación entre reservas y eventos

El sistema deberá diferenciar entre una reserva de espacio y un evento comercial.

Una reserva representa la ocupación de un espacio físico en una fecha y horario determinados.

Un evento representa una actividad comercial más completa, como un cumpleaños infantil o deportivo.

Ejemplos:

- Un alquiler común de cancha genera una reserva, pero no necesariamente un evento.
- Un cumpleaños deportivo genera un evento y utiliza la cancha como espacio físico.
- Un cumpleaños de salón genera un evento y utiliza el salón infantil como espacio físico.
- Una actividad de educación física escolar puede generar una reserva recurrente o individual de la cancha.

Regla principal:

- El sistema deberá evitar superposiciones por espacio físico, fecha y horario, independientemente de si se trata de una reserva simple o de un evento.

---

## 6.12 Caja diaria

El sistema permitirá consultar los ingresos diarios del complejo.

La caja diaria deberá mostrar:

- Fecha.
- Total ingresado.
- Ingresos por método de pago.
- Ingresos por área.
- Detalle de movimientos.
- Cliente o persona que pagó.
- Concepto del pago.
- Monto.
- Hora del movimiento.
- Usuario que registró el movimiento.

Cada movimiento de caja deberá registrar:

- Fecha y hora.
- Tipo de movimiento.
- Concepto.
- Monto.
- Método de pago.
- Cliente, si corresponde.
- Persona eventual, si corresponde.
- Descripción.
- Usuario responsable.

Tipos de movimiento:

- Ingreso.
- Egreso.
- Anulación.

Reglas:

- Cada pago, venta, reserva o evento deberá generar un movimiento de caja.
- Los movimientos de caja no deberán eliminarse.
- Si se anula una operación económica, deberá quedar registrado un movimiento de anulación.
- El sistema deberá permitir consultar caja por fecha.
- El sistema deberá permitir agrupar ingresos por método de pago.
- El sistema deberá permitir agrupar ingresos por área del complejo.

---

## 6.13 Informes

La versión 1.0 incluirá informes básicos.

Informes iniciales:

- Informe diario de ingresos totales.
- Informe de ingresos por área.
- Informe de ingresos por método de pago.
- Informe de deudas.
- Informe de productos más vendidos.
- Informe de pagos por cliente.
- Informe de ventas de confitería / cafetería.
- Informe de reservas.
- Informe de eventos.

El informe diario deberá permitir ver:

- Total ingresado en el día.
- Detalle de cada ingreso.
- Cliente o persona eventual que pagó.
- Concepto.
- Monto.
- Método de pago.
- Hora.
- Usuario que registró la operación.

---

## 6.14 Gráficos

El sistema deberá mostrar gráficos simples para facilitar la interpretación de los datos.

Gráficos iniciales:

- Ingresos por área.
- Ingresos por método de pago.
- Productos más vendidos.
- Deudas por actividad.
- Ingresos por día.

Los gráficos deberán ser simples, claros y comprensibles para usuarios sin experiencia técnica.

---

# 7. Usuarios y roles iniciales

La versión 1.0 deberá contemplar usuarios con permisos básicos.

## 7.1 Administrador

Puede realizar todas las acciones del sistema.

Funciones:

- Crear clientes.
- Modificar clientes.
- Dar de baja clientes.
- Registrar responsables.
- Registrar actividades.
- Inscribir clientes.
- Generar cuotas.
- Registrar pagos.
- Anular pagos.
- Registrar ventas.
- Anular ventas.
- Registrar reservas.
- Cancelar reservas.
- Registrar eventos.
- Cancelar eventos.
- Ver informes.
- Modificar precios.
- Administrar productos.
- Administrar usuarios.
- Consultar auditoría.

## 7.2 Encargado

Puede realizar la mayoría de las operaciones diarias del complejo.

Funciones:

- Registrar clientes.
- Modificar datos básicos de clientes.
- Registrar pagos.
- Registrar ventas.
- Registrar reservas.
- Registrar eventos.
- Consultar caja diaria.
- Consultar deudas.
- Consultar informes básicos.

Restricciones:

- No puede administrar usuarios.
- No puede modificar configuraciones críticas.
- No puede eliminar ni anular información crítica sin autorización, salvo que el administrador lo permita.

## 7.3 Empleado

Puede realizar operaciones simples.

Funciones:

- Registrar ventas.
- Consultar productos.
- Registrar pagos simples.
- Consultar reservas del día.

Restricciones:

- No puede ver informes completos.
- No puede modificar precios.
- No puede administrar usuarios.
- No puede anular operaciones críticas.

## 7.4 Consulta

Puede visualizar información, pero no modificar datos.

Funciones:

- Ver informes.
- Ver caja diaria.
- Consultar clientes.
- Consultar reservas y eventos.

Restricciones:

- No puede crear, modificar, anular ni eliminar información.

---

# 8. Módulos no incluidos en la Versión 1.0

La primera versión no incluirá:

- Facturación fiscal electrónica.
- Integración con AFIP.
- Envío automático de WhatsApp.
- Envío automático de emails.
- Aplicación móvil.
- Reservas online realizadas por clientes.
- Pagos online automáticos.
- Control avanzado de empleados.
- Control avanzado de proveedores.
- Control avanzado de gastos.
- Sistema contable completo.
- Integración bancaria.
- Impresión automática de tickets.
- Control avanzado de sueldos.
- Gestión avanzada de promociones.
- Notificaciones automáticas de vencimientos.

Estos módulos podrán agregarse en versiones futuras.

---

# 9. Reglas generales del sistema

Las reglas generales de la versión 1.0 serán:

- No se deberán borrar clientes definitivamente.
- Los clientes dados de baja deberán quedar como inactivos.
- No se deberán borrar pagos.
- Los pagos cargados por error deberán anularse.
- No se deberán borrar ventas con movimientos asociados.
- Las ventas cargadas por error deberán anularse.
- No se deberán borrar reservas con pagos asociados.
- No se deberán borrar eventos con pagos asociados.
- Toda operación económica deberá quedar registrada.
- Todo ingreso deberá tener fecha, hora, concepto, monto y método de pago.
- Todo pago deberá impactar en caja diaria.
- Toda venta deberá descontar stock.
- Toda anulación de venta deberá restaurar stock.
- No se podrá vender un producto sin stock suficiente.
- No se podrá registrar un pago con monto cero o negativo.
- No se podrá registrar una cuota con importe cero o negativo.
- No se podrá reservar un espacio en un horario ya ocupado.
- No se podrá crear una cuota duplicada para el mismo cliente, actividad y mes.
- Una cuota pagada completamente deberá quedar en estado pagada.
- Una cuota pagada parcialmente deberá quedar en estado parcial.
- Una cuota vencida con saldo pendiente deberá figurar como vencida.
- El sistema deberá permitir ventas a clientes registrados.
- El sistema deberá permitir ventas a personas no registradas.
- El sistema deberá permitir reservas y eventos de personas no registradas.
- El sistema deberá mostrar mensajes claros para usuarios sin experiencia.
- El sistema deberá evitar mostrar errores técnicos al usuario final.
- El sistema deberá registrar qué usuario realizó cada operación importante.

---

# 10. Reglas de anulación

La versión 1.0 deberá contemplar anulación de operaciones económicas sin pérdida de historial.

Reglas:

- Los pagos no se eliminan, se anulan.
- Las ventas no se eliminan, se anulan.
- Las reservas no se eliminan si tienen pagos asociados.
- Los eventos no se eliminan si tienen pagos asociados.
- Toda anulación deberá guardar fecha, usuario responsable y motivo.
- Si se anula una venta, el stock deberá restaurarse.
- Si se anula un pago asociado a una cuota, la deuda correspondiente deberá actualizarse.
- Si se anula un pago de reserva, el saldo pendiente deberá actualizarse.
- Si se anula un pago de evento, el saldo pendiente deberá actualizarse.
- Toda anulación económica deberá reflejarse en caja diaria como movimiento de anulación.

---

# 11. Estados generales del sistema

## 11.1 Estados de cliente

- Activo.
- Inactivo.

## 11.2 Estados de inscripción

- Activa.
- Suspendida.
- Finalizada.

## 11.3 Estados de cuota

- Pendiente.
- Parcial.
- Pagada.
- Vencida.
- Anulada.

## 11.4 Estados de pago

- Registrado.
- Anulado.

## 11.5 Estados de producto

- Activo.
- Inactivo.

## 11.6 Estados de venta

- Registrada.
- Anulada.

## 11.7 Estados de reserva

- Pendiente.
- Señada.
- Pagada.
- Cancelada.
- Realizada.

## 11.8 Estados de evento

- Pendiente.
- Señado.
- Pagado.
- Cancelado.
- Realizado.

---

# 12. Seguridad y privacidad

El sistema deberá proteger la información registrada, especialmente porque se almacenarán datos de menores de edad.

Reglas iniciales:

- Cada usuario deberá ingresar con usuario y contraseña.
- No todos los usuarios podrán acceder a todas las funciones.
- Los datos de clientes menores de edad deberán visualizarse solo por usuarios autorizados.
- Los pagos y movimientos de caja deberán guardar quién los registró.
- Las anulaciones deberán guardar motivo, fecha y usuario responsable.
- El sistema no deberá mostrar información sensible innecesaria.
- Las contraseñas no deberán guardarse como texto plano.
- La base de datos no deberá quedar expuesta directamente a internet.
- Si el sistema se publica en red o internet, deberá utilizar conexión segura.
- Se deberá evitar compartir usuarios entre varias personas.

---

# 13. Auditoría

El sistema deberá registrar información de auditoría en las operaciones importantes.

Datos mínimos de auditoría:

- Fecha de creación.
- Usuario que creó el registro.
- Fecha de última modificación.
- Usuario que modificó el registro.
- Fecha de anulación, si corresponde.
- Usuario que anuló, si corresponde.
- Motivo de anulación, si corresponde.

Operaciones importantes a auditar:

- Creación de clientes.
- Modificación de clientes.
- Baja de clientes.
- Registro de responsables.
- Creación de actividades.
- Cambios de precios.
- Inscripciones.
- Generación de cuotas.
- Registro de pagos.
- Anulación de pagos.
- Registro de ventas.
- Anulación de ventas.
- Registro de reservas.
- Cancelación de reservas.
- Registro de eventos.
- Cancelación de eventos.
- Movimientos de caja.

---

# 14. Respaldo de información

El sistema deberá permitir o facilitar la realización de copias de seguridad.

Reglas iniciales:

- Se deberá realizar backup periódico de la base de datos.
- Antes de actualizar el sistema, se deberá realizar una copia de seguridad.
- La información no deberá depender únicamente de una sola computadora.
- Se deberá probar que los backups puedan restaurarse correctamente.
- Las copias de seguridad deberán guardarse en un lugar distinto al equipo principal.
- El procedimiento de backup deberá estar documentado.

---

# 15. Criterios de aceptación de la Versión 1.0

La versión 1.0 se considerará terminada cuando el sistema permita completar correctamente el siguiente circuito:

1. Crear un cliente.
2. Crear un responsable.
3. Asociar el responsable al cliente.
4. Crear una actividad.
5. Inscribir el cliente a la actividad.
6. Generar una cuota mensual.
7. Registrar un pago parcial.
8. Consultar que la cuota quedó parcial.
9. Registrar el pago restante.
10. Consultar que la cuota quedó pagada.
11. Consultar deuda actualizada del cliente.
12. Registrar un producto.
13. Registrar una venta a cliente.
14. Registrar una venta a persona eventual.
15. Ver que el stock se descuenta correctamente.
16. Consultar productos con stock bajo.
17. Crear una reserva de cancha.
18. Registrar una seña de reserva.
19. Registrar el pago final de reserva.
20. Crear un evento infantil.
21. Registrar una seña de evento.
22. Registrar pago final de evento.
23. Consultar caja diaria.
24. Ver ingresos agrupados por método de pago.
25. Ver ingresos agrupados por área.
26. Consultar informe de deudas.
27. Consultar productos más vendidos.
28. Visualizar gráficos básicos.
29. Iniciar sesión con usuario y contraseña.
30. Verificar que cada usuario solo accede a las funciones permitidas.

---

# 16. Objetivo mínimo para considerar terminada la Versión 1.0

La versión 1.0 se considerará terminada cuando permita:

- Registrar clientes.
- Registrar responsables.
- Registrar actividades.
- Inscribir clientes a actividades.
- Generar cuotas mensuales.
- Registrar pagos.
- Registrar pagos parciales.
- Consultar deudas.
- Registrar productos.
- Registrar ventas de confitería / cafetería.
- Controlar stock.
- Registrar reservas de cancha.
- Registrar cumpleaños y eventos.
- Registrar señas y saldos.
- Consultar caja diaria.
- Consultar informes.
- Visualizar gráficos básicos de ingresos.
- Utilizar usuarios con permisos básicos.
- Auditar operaciones importantes.
- Realizar backup de la base de datos.

---

# 17. Requisitos de usabilidad

El sistema deberá ser simple y claro para usuarios sin experiencia técnica.

Reglas de usabilidad:

- Utilizar botones grandes y claros.
- Evitar términos técnicos innecesarios.
- Mostrar mensajes simples y comprensibles.
- Confirmar operaciones importantes antes de realizarlas.
- Mostrar estados con colores claros.
- Permitir búsquedas rápidas.
- Priorizar las acciones más usadas en la pantalla principal.
- Evitar pantallas sobrecargadas.
- Mostrar errores de forma amigable.

Ejemplos de textos recomendados:

- Cobrar cuota.
- Registrar venta.
- Nueva reserva.
- Nuevo evento.
- Ver deuda.
- Ver caja del día.
- Anular pago.
- Confirmar operación.

Ejemplos de textos no recomendados:

- Persistir entidad.
- Ejecutar transacción.
- Insertar registro económico.
- Actualizar recurso financiero.

---

# 18. Requisitos técnicos de la Versión 1.0

La versión 1.0 se desarrollará con las siguientes tecnologías:

- Java.
- Maven.
- Spring Boot.
- Spring Web.
- Spring Data JPA.
- MySQL.
- HTML.
- CSS.
- JavaScript.
- Chart.js para gráficos.
- Git para control de versiones.

Arquitectura esperada:

- Backend con API REST.
- Base de datos MySQL.
- Frontend web consumiendo la API.
- Separación por capas:
    - Controller.
    - Service.
    - Repository.
    - Entity.
    - DTO.
    - Mapper.
    - Exception.
    - Security.

---

# 19. Fuera de alcance técnico de la Versión 1.0

En la versión 1.0 no será obligatorio implementar:

- Aplicación móvil.
- Frontend con framework avanzado.
- Microservicios.
- Arquitectura distribuida.
- Integración con servicios externos.
- Pagos automáticos.
- Facturación electrónica.
- Envío automático de notificaciones.
- Sistema contable profesional.
- Dashboard avanzado en tiempo real.

---

# 20. Glosario inicial

## Cliente

Persona registrada en el sistema, principalmente niños que realizan actividades deportivas dentro del complejo.

## Responsable

Adulto asociado a un cliente menor de edad.

## Persona eventual

Persona que realiza una compra, reserva, pago o evento sin estar registrada como cliente.

## Actividad

Servicio ofrecido por el complejo, como fútbol, taekwondo, alquiler de cancha o salón infantil.

## Inscripción

Relación entre un cliente y una actividad.

## Cuota

Importe mensual generado por una inscripción.

## Pago

Registro de dinero recibido.

## Venta

Operación de venta de productos de confitería / cafetería.

## Reserva

Ocupación de un espacio físico en una fecha y horario determinados.

## Evento

Actividad comercial especial, como cumpleaños deportivo o cumpleaños de salón infantil.

## Caja diaria

Resumen de movimientos económicos registrados en una fecha determinada.

## Movimiento de caja

Registro individual de ingreso, egreso o anulación.

## Anulación

Acción que deja sin efecto una operación sin eliminar su historial.

---

# 21. Versiones futuras posibles

Luego de completar la versión 1.0, podrán evaluarse futuras mejoras:

- Facturación fiscal electrónica.
- Integración con AFIP.
- Envío automático de WhatsApp.
- Envío automático de emails.
- Recordatorios automáticos de vencimiento.
- Reservas online para clientes.
- Pagos online.
- App móvil.
- Exportación de informes a Excel.
- Generación de comprobantes PDF.
- Impresión de tickets.
- Control avanzado de empleados.
- Control de proveedores.
- Control de gastos.
- Control de compras de stock.
- Panel avanzado de estadísticas.
- Promociones y descuentos.
- Historial deportivo de alumnos.
- Asistencia a clases.
- Notificaciones internas.

---

# 22. Conclusión

La versión 1.0 del sistema tendrá como objetivo principal reemplazar el registro manual en papel por una herramienta web ordenada, segura y fácil de usar.

El sistema deberá permitir controlar clientes, actividades, cuotas, pagos, ventas, reservas, eventos, caja diaria, informes y gráficos.

La regla central del sistema será:

> Toda operación económica deberá quedar registrada con fecha, hora, concepto, monto, método de pago y usuario responsable.

Con esta base, el complejo podrá tener mayor control administrativo, reducir errores, consultar información histórica y tomar mejores decisiones sobre el funcionamiento diario del negocio.

---