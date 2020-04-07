# Reglas Integracion Creacion de Cesiones

#### \NDC\cesiones
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente
2. El servicio API recibira los datos desde el portal dejando estos en la **cola CESIONES**
3. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")

#### \CDI\cesiones
1. Debe extaer los datos desde la cola CESIONES
2. Debe extaer desde el JSON los siguientes datos
  -  [A] company_id, identificacion del cliente, rut
  -  [B] origen, origen de los datos entregados
3. Ejecutar servicio **tpgcETD\create** con las siguientes consideraciones:
  - tipodato : nombre de la cola ej: CESIONES
  - rut: incluye digito, sin puntos y guion ej: 122436756 ([A])
  - origen: ej: OCTA ([B])
  - datosjson: JSON completo de datos
  - El servicio create debe retornar ID del registro creado ([C])
4. Se debe ejecutar el servicio **\dtes\cesion** con las siguientes consideraciones:
  - Al servicio se debe enviar la informacion completa del documento cedido incorporando como dato adicional el LOG_ID con valor retornado al punto 3 (C)
5. Una vez terminado con la cesion de documentos se debe actualizar los datos **tpgcETD\update** a traves del ID rescatado en el punto 3 con los siguientes datos:
  - estado, error resultante de la ejecucion del servicio del punto 4
  - datosjson, en caso de que los datos json hallan cambiado
6. Eliminar el dato de la cola CESIONES

#### \dtes\cesion
  - Se consulta datos de documento a traves del servicio **\tpgcdte\select** con las siguientes consideraciones:
    - rutEmisor -> VENDEDOR
    - codigoTipoDoc -> TIPO_DOC
    - folio -> FOLIO_DOC
    - el servicio devolve:
      - dte_mto_ope, el monto total del documento (C) 
      - dte_num_ope, el numero de la operacion si esta asignada  (D)
  - Si el documento existe las tareas a realizar son
    - confirmar que la cesion del documento ha sido realizada hacia PFSA:
      - Si CEDENTE = VENDEDOR
      - Si CESIONARIO = 99501480-7
      - Si MNT_TOTAL = MNT_CESION = (C)  se acepta una diferencia no mayor a un 5%      
    - Ejecutar servicio **\tpgcdte\update\cesion** para actualizar datos con las siguientes consideraciones
      - rutEmisor -> VENDEDOR
      - codigoTipoDoc -> TIPO_DOC
      - folio  -> FOLIO_DOC
      - fecsec -> FCH_CESION
      - rutsec -> CESIONARIO
      - codlog -> LOG_ID
      - Si cumple la cesion del documento implica : estces = dte_vgt = SI
      - Si NO cumple la cesion del documento (CESIONARIO <> 99501480-7):estces = dte_vgt = NO

    - Si el documento queda en estado dte_vgt=NO y se encuentra asociado a un numero de operacion distinto a cero (D<>0) : Se debe enviar la alerta pertinente para que los distintos sistemas de curse (GVE / BPM) revisen la operación, recalculen en caso de ser necesario y posteriormente continuen con el proceso de curse. **Pendiente**
    - Si el documento queda en estado dte_vgt=SI y se encuentra asociado a un nuemro de operacion distinto a cero(D<>0): Se debe informar a los distintos sistemas de curse (GVE / BPM) que se ha cedido el documento por parte del cliente. **Pendiente**

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
  - la metodo retorna el ID del registro para control

#### \tpgcetd\update
Se debe ejecutar la accion de actualizar un registro en la tabla **tpgc_etd** segun ID con las siguientes consideraciones:
  - etd_jsn, enviado como parametro de entrada, si es blanco no se modifica
  - etd_fec_pcs, fecha y hora de termino de la actualizacion del registro
  - etd_est, enviado como parametro de entrada

#### \tpgcdte\select 
Se debe ejecutar la accion de seleccionar un registro en la tabla **tpgc_dte** con las siguientes consideraciones:
  - rutEmisor, enviado por parametros de entrada
  - codigoTipoDoc, enviado por parametro de entrada
  - folio, enviado por parametro de entrada
  - dte_vgt = SI
  - el metodo retorna los datos del o los documento encontrado.

#### \tpgcdte\update 
Se debe ejecutar la accion de actualiza un registro en la tabla **tpgc_dte** con las siguientes consideraciones:
  - rutEmisor, enviado por parametros de entrada
  - codigoTipoDoc, enviado por parametro de entrada
  - folio, enviado por parametro de entrada
  - fecsec, enviado por parametro de entrada
  - rutsec, enviado por parametro de entrada
  - estces, enviado por parametro de entrada
  - codlog, enviado por parametro de entrada
  - estreg, enviado por parametro de entrada
  - La actualizacion aplica a todos los registros encontrados

