# Reglas Integracion Cuenta y Contacto

#### \NDC\entidades\cuentas
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente
2. El servicio API recibira los datos desde el portal dejando estos en la **cola DATOSCLI**
3. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")
4. Se debe incorporar el Tipo de integracion al JSON recibido ("tipo": "CUENTA")

#### \NDC\entidades\contacto
1. El servicio API recibira los datos desde el portal dejando estos en la **cola DATOSCLI**
2. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")
3. Se debe incorporar el Tipo de integracion al JSON recibido ("tipo": "CONTACTO")

#### \CDI\ecliente\datos
1. Debe extaer los datos desde la cola DATOSCLI
2. Debe extaer desde el JSON los siguientes datos
  -  [A] origen, origen de los datos entregados
  -  [B] tipo, tipo de datos extraidos
  -  [C] rutcli, rut del cliente
3. Ejecutar servicio **tpgcETD\create** con las siguientes consideraciones:
  - tipodato : nombre de la cola ej: DATOSCLI
  - rut: incluye digito, sin puntos y guion ej: 122436756 ([C])
  - origen: ([A]) ej: OCTA
  - datosjson: JSON completo de datos
  - El servicio create debe retornar ID del registro creado
4. Ejecutar servicio **tpgcCLI\select** con las siguientes consideraciones:
  - rut ->  incluye digito, sin puntos y guion ej: 122436756 ([C])
  - el servicio retorna el ID del cliente creado en el core de clientes ([D])
  - Si cliente no existe ejecutar el punto 6 y 7
5. Ejecutar servicio transfieron el total de los datos del JSON 
  - Incorporar al JSON que se esta trasnfiriendo el ID retornado en el punto 4 (ej: idCliCore->([D]))
  - Si ([B]) = "CUENTA" servicio **cliente\cuenta** 
  - Si ([B]) = "CONTACTO"  servicio **cliente\contacto** 
6. Una vez terminado el proceso se debe actualizar los datos **tpgcETD\update** a traves del ID rescatado en el punto 3 con los siguientes datos:
  - estado, error resultante de la ejecucion del servicio del punto 4
  - datosjson, en caso de que los datos json hallan cambiado
7. Eliminar el dato de la cola cliente

#### \cliente\cuenta
  - El servicio se debe integrar con el servicio de **GVE-DatosCuenta** por rut del cliente
  - Debe extaer desde el JSON el dato idCliCore
  - Si idCliCore es distinto a blanco o cero se debe integrar con el servicio de **BMP-DatosCuenta**
  - Si idCliCore es igual a blanco o cero se debe integrar con el servicio de **\tpgccli\update\cuenta**

#### \cliente\Contacto
  - El servicio se debe integrar con el servicio de **GVE-DatosContacto** por rut del cliente
  - Debe extaer desde el JSON el dato idCliCore
  - Si idCliCore es distinto a blanco o cero se debe integrar con el servicio de **BMP-DatosContacto**
  - Si idCliCore es igual a blanco o cero se debe integrar con el servicio de **\tpgccli\update\contacto**

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

#### \tpgccli\update\cuenta
Se debe ejecutar la accion de actualizar un registro en la tabla **tpgc_cli** segun rut con las siguientes consideraciones:
  - cli_dat_cta -> JSON

#### \tpgccli\update\contacto
Se debe ejecutar la accion de actualizar un registro en la tabla **tpgc_cli** segun rut con las siguientes consideraciones:
  - cli_dat_cto -> JSON


#### \tpgcCLI\select\{rut}
Se debe ejecutar la accion de seleccionar un registro en la tabla **tpgc_cli** con las siguientes consideraciones:
  - cli_num_doc, (rut) enviado por parametros de entrada
  - cli_tip_doc, tipo de documento, para este caso tipo RUT
  - el metodo retorna los datos del cliente solicitado


