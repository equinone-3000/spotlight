# Reglas Integracion Creacion de DTES

#### ESTRUCTURA JSON DE ENTRADA

```json
{
  sheet_nbr: (INT) folio del documento,
  sender_tid: (STRING) rut del emisor (con guion, con DV, sin puntos),
  receiver_tid: (STRING) rut del receptor,
  executor_tid: (STRING) rut de quién recibe el pago,
  sender_name: (STRING) nombre de la empresa emisora,
  receiver_name: (STRING) nombre de la empresa receptora,
  executor_name: (STRING) nombre de la empresa quién recibe el pago,
  document_type: (INT) tipo de documento (33 factura, 34 factura exenta, 46 factura compra, 56 nota de credito, 61 nota de debito, 110 factura de exportacion, 111 nota credito exportacion, 112 nota debito exportacion),
  exempt_amount: (INT) monto exento de la factura,
  vat_amount: (INT) monto de iva,
  total_amount: (INT) monto total de la factura,
  net_amount: (INT) monto neto,
  hash_key: (STRING) identificador interno de la DTE en Octa,
  creation_date: (INT) unix timestamp fecha de creacion del documento,
  receival_date: (INT) unix timestamp fecha de recepción en el SII del documento,
  executor_date: (INT) unix timestamp fecha de última cesión
  related_docs: (ARRAY) arreglo de objetos, donde cada objeto representa un documento relacionado (nota de credito o debito)
  [
    {
      document_type: (INT) tipo de documento,
      sheet_nbr: (INT) folio,
      sender_tid: (STRING) rut del emisor (con guion, con DV, sin puntos),
      sender_name: (STRING) nombre de la empresa emisora,
      receiver_name: (STRING) nombre de la empresa receptora,
      total_amount: (INT) monto total del documento relacionado,
      vat_amount: (INT) monto de iva del documento relacionado,
      net_amount: (INT) monto neto del documento relacionado,
      creation_date: (INT) unix timestamp fecha de creacion del documento,
      receival_date: (INT) unix timestamp fecha de recepción en el SII del documento
    },
    ...
  ],
  events: (ARRAY) arreglo de objetos, donde cada objeto representa un evento del DTE
  [
    {
      state: (STRING) estado del evento, puede ser: emitted, received, paid, auto_confirmed, assumed_auto_confirmed, manual_confirmed, rejected, new_rejected_document ,
      date: (INT) unix timestamp fecha en que el evento ocurre,
      retrieved_date: (INT) unix timestamp fecha en que se obtiene el estado del evento
    }
  ]
}
```

#### \NDC\dtes
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente
2. El servicio API recibira los datos desde el portal dejando estos en la **cola DTES**
3. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el Portal Web de OCTA pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "OCTA")

#### \CDI\dtes
1. Debe extaer los datos desde la cola DTES

2. Debe extaer desde el JSON los siguientes datos para las variables
  -  [A] document_type, identificacion del tipo del documento
  -  [B] sender_tid,  rut de cliente emisor del documento (con guion, con DV, sin puntos),
  -  [D] origen, origen de los datos entregados
  -  [E] sheet_nbr, folio del documento
  -  [F] creation_dat, fecha de emision del documento
  -  [G] total_amount, Monto Total del documento
  -  [H] receiver_tid:, Rut del Receptor  del documento
  - Para document_type = 46 debemos invertir los datos del Emisor y del Recptor por lo tanto las variables :
  -  [B] receiver_tid
  -  [H] sender_tid

3. Ejecutar servicio **tpgcETD\create** con las siguientes consideraciones:
  - tipodato : nombre de la cola ej: DTES
  - rut: incluye digito, sin puntos y guion ej: 122436756 ([B])
  - origen: ej: OCTA ([D])
  - datosjson: JSON completo de datos
  - El servicio create debe retornar ID del registro creado ([J])  

4. Ejecutar servicio **tpgcCLI\select** con las siguientes consideraciones:
  - rut ->  incluye digito, sin puntos y guion ej: 122436756 ([B])
  - el servicio retorna el ID del cliente creado en el core de clientes ([K])

  - Si cliente no existe ejecutar el punto 7 y 8

5. Ejecutar servicio **tpgcDTE\select\origen** con las siguientes consideraciones:
  - rutEmisor -> ([B])
  - codigoTipoDoc -> ([A])
  - folio -> ([E])
  - origen -> ([D])
  - Determina si DTE existe o no en base de datos ([X])

6. Ahora debemos revisar las Notas referenciadas que pueda tener la factura
    - revisemos el apartado related_docs del JSON de entrada
    - iteremos por cada registro existente en el apartado
      - cuente ([L]) y sume ([M]) todos los related_docs/document_type =  61 el campo related_docs/total_amount
      - cuente ([N]) y sume ([O]) todos los related_docs/document_type =  56 el campo related_docs/total_amount


7. Si DTE no existe segun punto 5, se debe ejecutar el servicio **\dtes\nueva** con las siguientes consideraciones:
    - rutEmisor -> ([B])
    - codigoTipoDoc -> ([A])
    - rutReceptor -> ([H])
    - folio -> ([E]) 
    - fechaEmision -> ([F]) 
    - mntTotal -> ([G])
    - origen -> ([D])
    - Log_ID -> ([J]) 
    - idCliCore -> ([K])
    - mntTotNC -> ([M]) 
    - NumTotNC -> ([L]) 
    - mntTotND -> ([O]) 
    - NumTotND -> ([N])

   De los contario, se debe ejecutar el servicio **\dtes\notas** con las siguientes consideraciones:
    - rutEmisor -> ([B])
    - codigoTipoDoc -> ([A])
    - folio -> ([E]) 
    - mntTotNC -> ([M]) 
    - NumTotNC -> ([L]) 
    - mntTotND -> ([O]) 
    - NumTotND -> ([N])
    - Log_ID -> ([J])
    - NumOpe  -> dtenumope, valor retornado en punto 5

 
8. Una vez terminado con el DTE se debe actualizar los datos **tpgcETD\update** a traves del ID rescatado en el punto 3 con los siguientes datos:
  - estado, error resultante de la ejecucion del servicio del punto 4 o 5 segun sea el caso 
  - datosjson, en caso de que los datos json hallan cambiado

9. Eliminar el dato de la cola DTES

#### \dtes\nueva
  - Se debe ejecutar el servicio **\dtes\Mprecios** con las siguientes consideraciones:
    - rutEmisor -> enviado como parametro de entrada
    - rutReceptor -> enviado como parametro de entrada
    - El servicio retorna los datos comerciales a aplicar.
        - FecVcto ([L]) 
        - TasApl ([M])
        - FacAnt ([N])
  - Se debe ejecutar el servicio **\clientes\conacuerdo** con las siguientes consideraciones:
    - rutEmisor -> enviado como parametro de entrada
    - El servicio retorna los siguientes datos.
        - AcuPagador ([O]) 
        - SubLimite ([P])

  - Se debe ejecutar el servicio **\tpgcdte\create** con las siguientes consideraciones:
    - rutEmisor -> enviado como parametro de entrada
    - codigoTipoDoc -> enviado como parametro de entrada
    - rutReceptor -> enviado como parametro de entrada
    - folio-> enviado como parametro de entrada
    - fechaEmision -> enviado como parametro de entrada
    - mntTotal -> enviado como parametro de entrada
    - origen -> enviado como parametro de entrada
    - Log_ID -> enviado como parametro de entrada
    - idCliCore -> enviado como parametro de entrada
    - AcuPagador": ([O])
    - SubLimite": ([P])
    - FecVcto ->  ([L]) 
    - TasApl -> ([M])
    - FacAnt -> ([N])
    - MtoAbono  -> mntTotal
    - mntTotNC -> ([M]) 
    - NumTotNC -> ([L]) 
    - mntTotND -> ([O]) 
    - NumTotND -> ([N])

  - Ejecutar servicio **tpgcDTE\select\origen** con las siguientes consideraciones:
    - rutEmisor -> ([B])
    - codigoTipoDoc -> ([A])
    - folio -> ([E])
    - origen -> ([D])
    - Determina si DTE existe o no en base de datos, si existe se debe informar a Canal de Origen de la DTE la nueva oferta transferiando el JSON completo con los Datos. **OCTA Notificacion de Oferta **
        

#### \dtes\notas
  - Para Notas de Credito se debe ejecutar el servicio **\tpgcdte\update\credito** con las siguientes consideraciones:
    - rutEmisor -> enviado como parametro de entrada
    - codigoTipoDoc -> enviado como parametro de entrada
    - folio-> enviado como parametro de entrada
    - mntTotal -> mntTotNC, enviado como parametro de entrada
    - numTotal -> numTotNC, enviado como parametro de entrada
    - Log_ID -> enviado como parametro de entrada
  - Para Notas de Debito se debe ejecutar el servicio **\tpgcdte\update\debito** con las siguientes consideraciones
    - rutEmisor -> enviado como parametro de entrada
    - codigoTipoDoc -> enviado como parametro de entrada
    - folio-> enviado como parametro de entrada
    - mntTotal -> mntTotND, enviado como parametro de entrada
    - numTotal -> numTotND, enviado como parametro de entrada
    - Log_ID -> enviado como parametro de entrada

  - Si NumOpe es distinto a Cero se debe enviar la alerta pertinente para que los distintos sistemas de curse (GVE / BPM) revisen la operación, recalculen en caso de ser necesario y posteriormente continuen con el proceso de curse. **Pendiente**


#### \tpgcdte\update\credito
 Se debe ejecutar la accion de actualizar un registro en la tabla **tpgc_dte** con las siguientes consideraciones:
  - cli_num_doc -> rutEmisor, enviado por parametros de entrada
  - dte_tip_doc -> codigoTipoDoc, enviado por parametro de entrada
  - dte_fol -> folio, enviado por parametro de entrada
  - mntTotal, enviado por parametro de entrada
  - Se actulizan los siguientes datos:
    - dte_mto_ope -> dte_mto_tot + dte_mto_nd - mntTotal
    - dte_mto_abo -> dte_mto_ope * dte_fct_ant
    - dte_mto_nc -> mntTotal, enviado por parametro de entrada
    - dte_tot_nc -> numTotal, enviado por parametro de entrada
    - dte_log_ori -> Log_ID, enviado por parametro de entrada
    - Si dte_mto_ope <= 0 -> dte_vgt = "NO"

#### \tpgcdte\update\debito
Se debe ejecutar la accion de actualizar un registro en la tabla **tpgc_dte** con las siguientes consideraciones:
  - cli_num_doc -> rutEmisor, enviado por parametros de entrada
  - dte_tip_doc -> codigoTipoDoc, enviado por parametro de entrada
  - dte_fol -> folio, enviado por parametro de entrada
  - mntTotal, enviado por parametro de entrada
  - Se actulizan los siguientes datos:
    - dte_mto_ope -> dte_mto_tot + mntTotal - dte_mto_nc
    - dte_mto_abo -> dte_mto_ope * dte_fct_ant
    - dte_mto_nd -> numTotal, enviado por parametro de entrada
    - dte_tot_nd -> numTotal, enviado por parametro de entrada
    - dte_log_ori -> Log_ID, enviado por parametro de entrada

#### \dtes\Mprecios
  - Se integra con 2 servicios para calculo **Pendientes**
  - rutEmisor -> enviado como parametro de entrada
  - rutReceptor -> enviado como parametro de entrada
  - El servicio retorna los datos comerciales a aplicar.

#### \tpgcdte\create  
Se debe ejecutar la accion de crear un registro en la tabla **tpgc_dte** con las siguientes consideraciones:
- dte_id -> Dato autoincremental y unico
- cli_id -> idCliCore
- cli_num_doc -> rutEmisor
- dte_rut_rec -> rutReceptor
- dte_tip_doc -> codigoTipoDoc
- dte_fol -> folio
- dte_fec_emi -> fechaEmision
- dte_mto_tot -> mntTotal
- dte_log_ori -> Log_ID
- dte_cnl_ori -> origen
- dte_vgt -> SI
- dte_acu_pag -> AcuPagador
- dte_slt_dis -> SubLimite
- dte_fec_vto -> FecVcto
- dte_tsa_apl -> TasApl
- dte_fct_ant -> FacAnt
- dte_num_ope -> 0
- dte_mto_nc -> mntTotNC
- dte_tot_nc -> numTotNC
- dte_mto_nd -> mntTotND
- dte_tot_nd -> numTotND
- dte_mto_abo -> (mntTotal + mntTotND - mntTotNC) * FacAnt
- dte_mto_ope -> mntTotal + mntTotND - mntTotNC
- dte_est_ces -> NO
- dte_fec_ces -> null
- dte_rut_ces -> null
- dte_log_ces -> 0

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

#### \tpgcCLI\select\{rut}
Se debe ejecutar la accion de seleccionar un registro en la tabla **tpgc_cli** con las siguientes consideraciones:
  - cli_num_doc, (rut) enviado por parametros de entrada
  - cli_tip_doc, tipo de documento, para este caso tipo RUT
  - el metodo retorna los datos del cliente solicitado

