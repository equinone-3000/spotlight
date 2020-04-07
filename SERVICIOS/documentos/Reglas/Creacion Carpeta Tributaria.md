# Reglas Integracion Creacion Carpeta Tributaria

#### \NDC\ctributarias
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente
2. El servicio API recibira los datos desde el portal dejando estos en la **cola CARPETA**
3. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")

#### \CDI\ctributaria
1. Debe extaer los datos desde la cola CARPETA
2. Debe extaer desde el JSON los siguientes datos
  -  A link, ubicacion desde donde se debe desacargar la capeta tributaria
  -  B origen, origen de los datos entregados
  -  C rut, rut del cliente, incluye digito, sin puntos y guion
3. Ejecutar servicio **tpgcETD\create** con las siguientes consideraciones:
  - tipodato : nombre de la cola ej: CARTERA
  - rut: incluye digito, sin puntos y guion ej: 122436756 (C) 
  - origen: ej: OCTA (B)
  - datosjson: JSON completo de datos (A)
  - El servicio create debe retornar ID del registro creado
4. Se debe ejecutar el servicio **\clientes\doctos\ctributaria** con las siguientes consideraciones:
  - link -> (A)
  - El servicio retorna una estructura json con los datos asociados a la Carpeta Tributaria
5. Una vez terminado el procesamiento de la Carpeta Tributaria se debe actualizar los datos **tpgcETD\update** a traves del ID rescatado en el punto 3 con los siguientes datos:
  - estado, error resultante de la ejecucion del servicio del punto 4
  - datosjson, en caso de que los datos json hallan cambiado segun punto 4
6. Eliminar el dato de la cola CARPETA

#### \clientes\doctos\ctributaria
  - El servicio debe orquestar la descarga del documento desde el link infromado como parametro y el servicios de analisis de SAFE.
  - Con el link se descarga documento en formato de BASE64.
  - Se carga documento en formato de BASE64 en Servicio de Analisis de SAFE **(Servicio Pendiente)** es devuelve informacion de la carpeta tributaria en formato JSON.  
  - Se retorna informacion en formatoa JSON de carpeta tributaria

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

