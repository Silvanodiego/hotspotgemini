🛰️ DOCUMENTACIÓN TÉCNICA - HOTSPOT CyS TECNO

Este documento representa la única fuente de verdad para la lógica de red y flujo de cobro del sistema de Hotspot.
1. Arquitectura del Sistema

    Gateway (Silvano): Notebook local (192.168.50.1) que controla el acceso físico, las reglas de iptables y el portal cautivo en el puerto 5000.

    Cajero (VPS): Servidor remoto (173.249.54.14) que procesa los pagos de Mercado Pago y gestiona la base de datos de transacciones aprobadas.

2. Flujo de Usuario y Captura de MAC

Para evitar la escritura manual de IDs, el sistema utiliza el siguiente "puente":

    QR Físico: Ubicado en el local. Apunta exclusivamente a: [http://192.168.50.1:5000/vender](http://192.168.50.1:5000/vender).

    Captura de MAC: Al entrar a la ruta /vender, la Notebook identifica la IP del cliente, busca su dirección MAC en la tabla ARP y genera un ID único (CID).

    Salto Invisible: La Notebook redirige al cliente al VPS pasando el ID: [http://173.249.54.14/planes?cid=ID_GENERADO](http://173.249.54.14/planes?cid=ID_GENERADO).

    Identificación: El VPS recibe el cid y muestra los botones de pago vinculados a ese ID específico.

3. Sincronización y Habilitación

    Servicio en Notebook: Un hilo secundario (monitor_loop) consulta cada 5 segundos al VPS: GET [http://173.249.54.14/api/check/CID](http://173.249.54.14/api/check/CID).

    Acción tras Pago: Si el VPS responde "ok", la Notebook ejecuta los comandos de iptables para liberar el tráfico de la MAC asociada a ese CID por el tiempo comprado.

4. Reglas de Red Críticas (Walled Garden)

Para que el cliente pueda pagar aunque no tenga tiempo de navegación, se deben mantener siempre activas estas reglas en Silvano:

    Acceso al VPS: sudo iptables -I FORWARD -d 173.249.54.14 -j ACCEPT.

    Mercado Pago: Se deben permitir los dominios de mercadopago.com para evitar que el proceso de pago se corte a mitad de camino.

5. Rutas Definidas
En Notebook (Silvano):

    /: Portal de inicio (ofrece 5 min gratis).

    /conectar: Activa los 5 min iniciales.

    /vender: Captura MAC y redirige al VPS con el CID.

En VPS:

    /planes: Recibe el CID y muestra los planes de precio.

    /pagar: Genera la preferencia de Mercado Pago.

    /webhook: Recibe la confirmación de pago desde Mercado Pago.

    /api/check/<cid>: API que informa a Silvano sobre el estado del pago.

💡 Instrucción para Gemini:

    "Antes de sugerir cualquier cambio en el código o en las reglas de firewall, consultá este archivo. No modifiques la IP del VPS (173.249.54.14), no cambies el puerto de Flask (5000) y respetá que el QR apunta siempre a la Notebook local." Si leiste todo poneme aplausos al final de la respuesta.
