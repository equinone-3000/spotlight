# Reglas Integracion con Gestor de Contratos

#### \Clientes\{rut}\Contratos\{estado}

1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente.
2. El servicio se debe integrar con el **WSGestorDeContratos** atraves del rut del cliente y parametro de estado declarados en el Path del servicio, donde estado puede tomar los siguientes valores:
   - firmados -> Metodo obtenerContratosFirmados
   - nofirmados -> Metodo obtenerContratosPorFirmar
   - todos -> union de los datos de los metodos anteriores identificando claramente el estado del contrato  


#### \Clientes\{rut}\ReprLegales

1. El servicio API a travez del API-KEY debe identificar si quien lo esta invocando tiene o no autorizacion de hacerlo
   - Si el usuario NO tiene acceso retornar error de acceso no autorizado (401)
   - Si el usuario SI tiene acceso continua con el flujo siguiente.
2. El servicio se debe integrar con el **WSGestorDeContratos** atraves del rut del cliente invocando el metodo obtenerReprLegales.

3. El link devuelto debe tener una estructura para descarga de archivos a traves del servicio:
    - \Clientes\{rut}\Contratos\{pdf} -> \Clientes\1-9\Contratos\escritura.pdf


#### \Clientes\{rut}\Contratos\{pdf}

Este servicio debe permitir a traves del ID del PDF ir a descargar el documento al gestor de contratos y entregarlo al portal para descarga.



