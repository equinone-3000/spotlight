# Reglas Integracion con Gestor de Documental

#### \Clientes\{rut}\doctos\{tipo}
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente.

2. Ejecutar servicio **tpgcCLI\select** con las siguientes consideraciones:
  - rut ->  incluye digito, sin puntos y guion ej: 122436756 ([B])
  - el servicio retorna el ID del cliente creado en el core de clientes ([K])
  - Si cliente no existe retornar error de cliente no Existe (999) y terminar la ejecucion

3. Si el cliente existe solicitar a BPM la documentacion segun parametro tipo, donde tipo puede tomar los valores de:

  - LEGAL : Obtener listado de documentos (set legal) que se le debe solicitar al cliente
  - FACTORING : Obtener listado de documentos requeridos por la operación de FACTORING.
  - CONFIRMING : Obtener listado de documentos requeridos por la operación de CONFIRMING.

4. El link devuelto para descarga del documento almaceando debe tener una estructura para descarga de archivos a traves del servicio: - \Clientes\{rut}\documental\{pdf} -> \Clientes\1-9\documental\012938siuejeu90982983.pdf

#### \Clientes\{rut}\documental\{pdf}
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente.

2. Este servicio debe permitir a traves del ID del PDF ir a descargar el documento al administrador documental y entregarlo al portal para descarga.


#### \NDC\documentos
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente

2. El servicio API recibira los datos desde el portal dejando estos en la **cola DOCUMENTOS**

3. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")


#### \CDI\documentos
1. Debe extaer los datos desde la cola DOCUMENTOS

2. Debe extaer desde el JSON los siguientes datos
  - (A) origen, origen de los datos entregados
  - (B) rut, rut del cliente, incluye digito, sin puntos y guion
  - (C) tipDocumentos, tipo del documento
  - (D) numoperacion, numero de la operacion

3. Ejecutar servicio **tpgcETD\create** con las siguientes consideraciones:
  - tipodato : nombre de la cola ej: DOCUMENTOS
  - rut: incluye digito, sin puntos y guion ej: 122436756 (B) 
  - origen: ej: OCTA (A)
  - datosjson: JSON completo de datos
  - El servicio create debe retornar ID del registro creado

4. por cada documento en el listado de documentos existente el JSON se debe:
  - Con el link se descarga documento en formato de BASE64.
  - (E) id, del sub-tipo de documento
  - Se ejecuta servicio **DOCUMENTAL CargaDocumento** enviado los siguientes datos
      - origen -> (A)
      - rutcli -> (B) 
      - tipDoc -> (C) 
      - numope -> (D) 
      - idTip  -> (E) 
      
5. Una vez terminado el proceso de carga de documentos debe actualizar los datos **tpgcETD\update** a traves del ID rescatado en el punto 3 con los siguientes datos:
  - estado, error resultante de la ejecucion del servicio del punto 4
  - datosjson, en caso de que los datos json hallan cambiado segun punto 4

6. Eliminar el dato de la cola DOCUMENTOS


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


#### \Notificar\Documento
Ejecuta servicios **BPM NotificacionCargaDocumento** Notificando Carga de documento asociada al cliente




