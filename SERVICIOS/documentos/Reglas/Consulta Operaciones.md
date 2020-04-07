# Reglas Integracion para Consulta de Operaciones

#### \Clientes\{rut}\Operaciones\{estado}
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente.

2. Ejecutar servicio **tpgcCLI\select** con las siguientes consideraciones:
  - rut ->  incluye digito, sin puntos y guion ej: 122436756 
  - el servicio retorna el ID del cliente creado en el core de clientes (A)
  - Si cliente no existe retornar error de cliente no Existe (999) y terminar la ejecucion
  - Este servicio solo sera requerido para integracion con **BPM ConsultaOperaciones**

3. Ejecutar servicio **GVE -ConsultaOperaciones** por Rut de Cliente y estado de operacion **PENDIENTE** con las siguientes consideraciones:
- rut ->  incluye digito, sin puntos y guion dato viene como parametro
- estado -> dato viene como parametro, sus posibles valores son enCurse / Rechazadas / Abonadas

4. Ejecutar servicio **BPM -ConsultaOperaciones** por ID de Cliente y estado de operacion **PENDIENTE**
- idcli -> ID del cliente creado en el core de clientes (A)
- estado -> dato viene como parametro, sus posibles valores son enCurse / Rechazadas / Abonadas

5. Por cada operacion rescata en los puntos 3 o 4 debe ejecutar servicio **\tpgcEtpOpe\Select\{operacion}** con las siguientes consideraciones:
- operacion, numero de operacion a consultar, dato viene por parametro


#### \tpgcCLI\select\{rut}
Se debe ejecutar la accion de seleccionar un registro en la tabla **tpgc_cli** con las siguientes consideraciones:
  - cli_num_doc, (rut) enviado por parametros de entrada
  - cli_tip_doc, tipo de documento, para este caso tipo RUT
  - el metodo retorna los datos del cliente solicitado


#### \tpgcEtpOpe\Select\{operacion}
Se debe ejecutar la accion de seleccionar los registro en la tabla **tpgc_etp_ope** relacionada con tabla **tpgc_etp**  con las siguientes consideraciones:
  - eop_num_ope, numero de la operacion enviado por parametros de entrada
  - el metodo retorna los datos de las etapas asociadas a la operacion






