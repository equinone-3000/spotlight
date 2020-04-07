# Reglas Integracion Creacion Entidad

#### \NDC\entidades
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente
2. El servicio API recibira los datos desde el portal dejando estos en la **cola CLIENTE**
3. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")

#### \CDI\ecliente
1. Debe extaer los datos desde la cola CLIENTE
2. Debe extaer desde el JSON los siguientes datos
  -  [A] contribuyente\rut, Rut del cliente
  -  [B] contribuyente\dv, Digito Verificador asociado al Rut del cliente,
  -  [C] contribuyente\origen, origen de los datos entregados
  -  [D] contribuyente\paisCodigo, pais de origen de la empresa (997 - CHILE)
  -  [E] contribuyente\personaEmpresa, tipo empresa (PERSONA o EMPRESA)
  -  [F] contribuyente\nombres 
  -  [G] contribuyente\apellidoPaterno 
  -  [H] contribuyente\apellidoMaterno
  -  [I] contribuyente\razonSocial

3. Ejecutar servicio **tpgcETD\create** con las siguientes consideraciones:
  - tipodato : nombre de la cola ej: CLIENTE
  - rut: incluye digito, sin puntos y guion ej: 122436756 ([A]&[B])
  - origen: ej: OCTA
  - datosjson: JSON completo de datos
  - El servicio create debe retornar ID del registro creado
4. Se debe ejecutar el servicio **\clientes\nuevo** con las siguientes consideraciones:
  - tipodoc, 1 para tipo de documento RUT
  - rut -> [A]&[B]
  - paisCodigo -> [D], si es null considerar 997
  - personaEmpresa -> [E] 
  - Si personaEmpresa = PERSONA, identificacion -> [F]&[G]&[H]
  - Si personaEmpresa = EMPRESA, identificacion -> [I]
5. Una vez terminado con la creacion de cliente se debe actualizar los datos **tpgcETD\update** a traves del ID rescatado en el punto 3 con los siguientes datos:
  - estado, error resultante de la ejecucion del servicio del punto 4
  - datosjson, en caso de que los datos json hallan cambiado
6. Eliminar el dato de la cola cliente

#### \clientes\nuevo
  - El servicio debe orquestar la creacion de cliente o entidad en GVE, Core Cliente y base de datos local
  - Se crea cliente en GVE a traves del WS PresimulaWS metodo **CrearCliente**, donde
    - rut -> enviado como parametro de entrada 
    - rsocial -> enviado como parametro de entrada ([identificacion])
  - Se crea cliente en Core Cliente a traves de API **CrearCliente**, devuelve ID de cliente creado.  
  - Se crea cliente en bases de datos local a traves del servicio **tpgccli\create** con las siguientes consideraciones
    - idCliCore, generado por la creacion en Core Cliente.
    - personaEmpresa, enviado como parametro de entrada
    - identificacion, enviado como parametro de entrada
    - paisCodig, enviado como parametro de entrada
    - tipodoc, enviado como parametro de entrada
    - rut, enviado como parametro de entrada

#### \tpgcetd\create
Se debe ejecutar la accion de crear un registro en la tabla **tpgc_etd** con las siguientes consideraciones:
  - etd_id, correlativo autoincremental
  - etd_rut, enviado como parametro de entrada
  - etd_org, enviado como parametro de entrada
  - etd_tip, enviado como parametro de entrada
  - etd_jsn, enviado como parametro de entrada
  - etd_fec_ing, fecha y hora de creacion del registro
  - etd_fec_pcs, fecha y hora de creacion del registro
  - etd_est, dato fijo, "en proceso..."
  - la creacion retorna el ID del registro para control

#### \tpgcetd\update
Se debe ejecutar la accion de actualizar un registro en la tabla **tpgc_etd** segun ID con las siguientes consideraciones:
  - etd_jsn, enviado como parametro de entrada, si es blanco no se modifica
  - etd_fec_pcs, fecha y hora de termino de la actualizacion del registro
  - etd_est, enviado como parametro de entrada

#### \tpgccli\create
Se debe ejecutar la accion de crear un registro en la tabla **tpgc_cli** segun ID con las siguientes consideraciones:
  - validar si el regitro existe antes de crearlo
  - cli_id, enviado como parametro de entrada
  - cli_etd_tip, enviado como parametro de entrada
  - cli_rzn_scl, enviado como parametro de entrada
  - cli_pai, enviado como parametro de entrada
  - cli_tip_doc, enviado como parametro de entrada
  - cli_num_doc, enviado como parametro de entrada


