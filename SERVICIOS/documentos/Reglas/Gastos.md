# Reglas Integracion para Gastos

#### \Clientes\{rut}\gastos\{producto}\portal
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente.

2. Ejecutar servicio **\Cliente\{rut}\Datos** para busqueda de informacion de Mora y Comisiones con las siguientes consideraciones:
  - rut ->  incluye digito, sin puntos y guion ej: 122436756 
  - Si Servicio retorna error no continua y terminar la ejecucion

3. Ejecutar servicio **tpgcGTO\select** con las siguientes consideraciones:
  - gto_prd -> producto consultado (FACTORING / CONFIRMING)
  - gto_suc -> portal, este dato es fijo para el portal
  - el servicio retorna los gastos asociados al producto para el portal web 

4. Armar Json de Salida y devolver


#### \Cliente\{rut}\Datos
1. Ejecutar servicio **tpgcCLI\select** con las siguientes consideraciones:
  - rut ->  incluye digito, sin puntos y guion ej: 122436756 
  - el servicio retorna el ID del cliente creado en el core de clientes (A)
  - Si cliente no existe retornar error de cliente no Existe (999) y terminar la ejecucion
  - Este servicio solo sera requerido para integracion con **BPM ConsultaDatosCliente**

2. Ejecutar servicio **GVE -ConsultaDatosCliente** por Rut de Cliente para retorno dedatos de mora y/o comision **PENDIENTE**

3. Ejecutar servicio **BPM -ConsultaDatosCliente** por Rut de Cliente para retorno dedatos de mora y/o comision **PENDIENTE**


#### \tpgcGTO\select
Se debe ejecutar la accion de seleccionar un registro en la tabla **tpgc_gto** con las siguientes consideraciones:
  - gto_prd, enviado por parametros de entrada
  - gto_suc, enviado por parametro de entrada, este dato es opcional, en caso de no llegar debe listar todos los gatos asociados al producto independiente de la sucursal
  - el metodo retorna los datos del o los documento encontrado.


#### \tpgcCLI\select\{rut}
Se debe ejecutar la accion de seleccionar un registro en la tabla **tpgc_cli** con las siguientes consideraciones:
  - cli_num_doc, (rut) enviado por parametros de entrada
  - cli_tip_doc, tipo de documento, para este caso tipo RUT
  - el metodo retorna los datos del cliente solicitado

