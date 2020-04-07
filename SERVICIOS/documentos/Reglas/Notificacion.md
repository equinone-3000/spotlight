# Reglas Integracion para Notificacion

#### \Notificacion\{operacion}\ABONO
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente

2. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el CORE BPM pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "BPM")

3. Ejecutar servicio **\tpgcEtpOpe\create** con las siguientes consideraciones:
  - etpcod -> ABO
  - eopnumope -> {operacion}, viene como parametro de entrada
  - eopfechor -> fechaHora, viene como parametro de entrada
  - eopest -> "REALIZADO"
  - eopfol_dte -> 0
  - eopobs -> JSON completo de datos

4. Ejecutar servicio **OCTA NotificaAbono** informando de la notificacion del abono **Pendiente**


#### \Notificar\{operacion}\cambios\{respuesta}
1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente

2. Debera identificar a travez del API-KEY quien esta enviando los datos (origen), hoy es el CORE BPM pero se quiere que a futuro se incorporen otros canales. Por tanto, se debe incorporar el dato de origen al JSON recibido ("origen": "BPM")

3. Ejecutar servicio **\tpgcEtpOpe\create** con las siguientes consideraciones:
  - etpcod -> MOD
  - eopnumope -> {operacion}, viene como parametro de entrada
  - eopfechor -> fechaHora del sistema
  - eopest -> {respuesta}, viene como parametro de entrada
  - eopfol_dte -> 0
  - eopobs -> JSON completo de datos

4. Ejecutar servicio **BPM NotificaModOpera** informando la respuesta al del cambio de la operacion **Pendiente**

5. Ejecutar servicio **GVE NotificaModOpera** informando la respuesta al del cambio de la operacion **Pendiente**


