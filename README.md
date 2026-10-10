# Master

App personal para llevar la tarjeta de crédito Mastercard: lee los avisos de compra que el banco envía por correo, guarda fecha, comercio, monto y tarjeta (Titular o Adicional), y calcula el cupo disponible a partir de la deuda pendiente.

Es un registro personal. No es la app oficial del banco ni está conectada a la tarjeta.

## Qué hace

- **Disponible, utilizado y total a pagar**: se parte ingresando a mano la deuda pendiente (el cupo utilizado que muestra el banco). Con el cupo de $5.000.000, la app muestra el saldo disponible, el monto utilizado y el total a pagar, que deja fuera las cuotas que vencen después de este mes. Con **Ajustar** se vuelve a cuadrar la deuda o se cambia el cupo.
- **Lectura automática**: una automatización de Atajos envía cada correo del banco, apenas llega a Mail, a un buzón privado en GitHub. La app lee ese buzón al abrirse y agrega solo las compras con tarjeta de crédito que falten. Un mismo aviso nunca se registra dos veces.
- **Titular o Adicional**: el aviso trae los últimos cuatro dígitos de la tarjeta. `****9908` se registra como Titular y `****4526` como Adicional.
- **Compra, cargo y abono a mano**: cada compra y cada cargo (−) restan del disponible; cada abono (+) suma. Sirve para los cobros de la tarjeta (comisiones, intereses, seguros) y para los pagos o devoluciones.
- **Compras en cuotas**: se ingresan con el monto total, el número de cuotas y el mes en que se paga la cuota 1. Una sección aparte muestra cada compra con sus cuotas pagadas, la cuota de este mes y las que vencen después. Una compra leída de un aviso se convierte en compra en cuotas al tocarla e indicar sus cuotas.
- **Totales por tarjeta**: cuánto suman las compras del Titular y del Adicional, más los cargos y los abonos.
- **Extracto**: un botón genera el detalle de todos los movimientos desde la deuda inicial, con el utilizado, las cuotas que vencen después y el total a pagar. Se copia como texto, se comparte o se exporta a Excel.
- **Borrar un registro**: al tocar un movimiento se corrige o se elimina. Hay unos segundos para deshacer.
- **Datos y respaldo**: copia todos los datos como texto y los restaura desde ahí.

## Cómo se calcula

- **Utilizado** = deuda inicial + compras + cargos − abonos, contando los movimientos desde la fecha de esa deuda.
- **Disponible** = cupo − utilizado.
- Los movimientos anteriores a la fecha de la deuda inicial ya están dentro de ella: quedan como historial y no cambian el saldo.
- **Total a pagar** = utilizado − cuotas que vencen después de este mes. Si no hay compras en cuotas, es igual al utilizado.
- Una compra en cuotas ocupa el cupo por su monto total desde el día de la compra, igual que en el banco, y se libera con cada abono. Por eso el disponible calza con el del banco aunque el total a pagar sea menor.
- Las cuotas se cuentan por mes calendario. La cuota 1 se paga en el mes indicado al anotar la compra (por omisión, el mes siguiente a la compra) y las demás, una por mes. Las de meses anteriores cuentan como pagadas, la de este mes entra en el total a pagar y las de meses siguientes quedan fuera hasta que llegue su mes.
- Ejemplo: una compra en 2 cuotas con la cuota 1 en octubre. En octubre, el total a pagar incluye la cuota 1 y deja fuera la cuota 2, que entra en noviembre.
- Las cuotas no se marcan a mano y no cambian el saldo. El disponible sube cuando se anota el abono.
- El valor de cada cuota es el total dividido en partes iguales, sin intereses. Si la compra tiene interés, conviene ingresar como monto total lo que se va a pagar en total.
- Una compra en cuotas anterior a la deuda inicial se anota con su fecha real: queda como historial, no cambia el saldo y sus cuotas futuras igual se descuentan del total a pagar.

## Exportar el extracto a Excel

En el extracto, **Exportar a Excel** genera el archivo `Extracto Master aaaa-mm-dd.xlsx` con tres hojas:

- **Resumen**: cupo, utilizado, disponible y total a pagar, más el detalle que lleva de la deuda inicial al total (compras del Titular y del Adicional, cargos, abonos y cuotas que vencen después).
- **Movimientos**: una fila por movimiento, con número, fecha, hora, descripción, tipo, tarjeta, cuotas y monto. Los montos van como deuda (compras y cargos en positivo, abonos en negativo), así que la suma de la columna es el utilizado. Trae filtros y la primera fila fija.
- **Cuotas**: una fila por compra en cuotas, con el valor de la cuota, el mes de la primera y de la última, las pagadas, la de este mes y las que vencen después. Solo aparece si hay compras en cuotas.

En el iPhone el archivo se entrega por la hoja de compartir, para guardarlo en Archivos, abrirlo en Excel o Numbers, o enviarlo. En un computador se descarga. El archivo se arma en el dispositivo, sin conexión y sin enviar los datos a ningún servicio.

## Formato del aviso

El lector reconoce el aviso de compra con tarjeta de crédito del Banco de Chile (remitente `enviodigital@bancochile.cl`):

> Te informamos que se ha realizado una compra por $7.930 con Tarjeta de Crédito ****4526 en RedGloba*SU CASERO el 30/07/2026 09:34. Revisa Saldos y Movimientos en App Mi Banco o Banco en Línea.

De cada aviso toma el monto, la tarjeta, el comercio, la fecha y la hora. Funciona con el texto plano o con el correo en HTML, y también si el correo llega codificado (con `=C3=A9` y líneas cortadas con `=`, o en base64).

La compra se reconoce como de la tarjeta por su **terminación** (`9908` o `4526`), no por la palabra «Crédito»: según cómo Atajos lea el correo, esa palabra puede llegar con el acento dañado (`CrÃ©dito`, `Cr?dito`).

Deja fuera:

- las compras con cargo a la cuenta (esas las registra FAN);
- las compras en dólares, porque no usan el cupo en pesos;
- las compras con otra tarjeta u otro medio de pago, es decir, cuya terminación no sea `9908` ni `4526`.

Si el banco reemplaza una tarjeta y cambia su terminación, se agrega la nueva en la tabla `CARDS`, al comienzo del código de `index.html`.

## Capturar los correos de la app Mail

iOS no permite que una app web lea el correo ni reciba datos de otras apps mientras está cerrada. Para que las compras aparezcan sin hacer nada, cada aviso pasa por un buzón en internet:

1. El correo del banco llega a Mail.
2. Una automatización de **Atajos** lo envía al buzón: un repositorio **privado** de GitHub, donde cada correo queda como un *issue*.
3. La app lee el buzón al abrirse (y cada vez que vuelves a ella) y agrega las compras nuevas.

### Si ya usas FAN

No hay que crear nada. El buzón (`fan-avisos`) y la automatización de Atajos son los mismos: la automatización envía todos los correos del banco, FAN toma las compras de la cuenta y Master, las de la tarjeta de crédito.

La única condición es que la automatización filtre **solo por remitente**. Si tiene una condición de asunto (por ejemplo, «cargo en cuenta»), hay que quitarla: los avisos de la tarjeta de crédito llegan con otro asunto y no pasarían al buzón.

1. Abre FAN, entra a **Buzón y atajo** y toca **Copiar autorización**.
2. Abre Master y toca **Conectar buzón**.
3. Revisa el repositorio (`tu-usuario/fan-avisos`) y pega la autorización en **Token de acceso**. Se puede pegar tal cual, con la palabra `Bearer` delante.
4. Toca **Probar y conectar**.

### Desde cero

**1. Crear el buzón en GitHub**

1. Crea un repositorio nuevo, **privado**, llamado `fan-avisos`.
2. Ve a **Settings → Developer settings → Personal access tokens → Fine-grained tokens** y toca **Generate new token**.
3. Ponle un nombre y elige la expiración. Cuando el token venza habrá que crear otro y volver a conectarlo.
4. En **Repository access** elige **Only select repositories** y marca `fan-avisos`.
5. En **Permissions**, da a **Issues** el acceso **Read and write**.
6. Toca **Generate token** y copia el token. GitHub lo muestra una sola vez.

**2. Conectar la app**

1. En Master, toca **Conectar buzón**.
2. Escribe el repositorio (`tu-usuario/fan-avisos`) y pega el token.
3. Toca **Probar y conectar**. La hoja **Buzón y atajo** muestra entonces los datos para el paso siguiente, con botones para copiarlos.

**3. Crear la automatización en Atajos**

1. Abre **Atajos**, entra a **Automatización** y toca **+**.
2. Elige **Correo electrónico** y en **Remitente** escribe `enviodigital@bancochile.cl`. Deja vacíos **Asunto** y los demás campos.
3. Marca **Ejecutar inmediatamente**, toca **Siguiente** y elige **Nueva automatización en blanco**.
4. Toca **Agregar acción**, busca «URL» y elige **Obtener contenido de URL**. En la URL pega `https://api.github.com/repos/tu-usuario/fan-avisos/issues`.
5. Toca **Mostrar más** y en **Método** elige **POST**.
6. En **Encabezados**, agrega uno nuevo con la clave `Authorization` y el valor `Bearer ` seguido del token.
7. En **Solicitar cuerpo** deja **JSON** y agrega dos campos de tipo **Texto**: `title` con el valor `Aviso`, y `body` con la variable **Entrada del atajo**.
8. Toca **Listo**.

### Notas

- Los avisos deben llegar a una cuenta agregada en la app Mail del iPhone. Los nombres de las opciones pueden variar un poco según la versión de iOS.
- El token da acceso solo a los *issues* de ese repositorio privado. Queda guardado en Atajos y, dentro de la app, solo en el dispositivo; no entra en los respaldos.
- En **Buzón y atajo** la app muestra los últimos correos que llegaron al buzón y qué hizo con cada uno, y avisa si llegaron compras de otra tarjeta de crédito.
- Para comprobar que Atajos funciona, mira la pestaña **Issues** del repositorio después de una compra: debe aparecer un *issue* nuevo con el texto del correo.
- Una compra eliminada en la app no vuelve a aparecer al leer el buzón. Para recuperarla, se pega su aviso a mano con **Pegar un aviso**.
- Las compras del buzón anteriores a la fecha de la deuda inicial entran como historial y no cambian el saldo.
- La lectura del buzón funciona en la app publicada en GitHub Pages. Dentro de Claude, la página no puede llamar a GitHub y las compras se agregan pegando el aviso.

### Si llega un correo y la compra no aparece

1. Toca **Actualizar**. Al volver a la app, el buzón se relee solo si pasaron más de dos minutos desde la última lectura.
2. Abre **Buzón y atajo** y mira **Últimos correos del buzón**. Ahí están los últimos correos que recibió el buzón, del más reciente al más antiguo, con lo que la app hizo con cada uno.
3. Si el correo no está en la lista, no llegó al buzón: el problema está en la automatización de Atajos (tiene una condición de asunto que lo dejó fuera, no se ejecutó o GitHub rechazó el envío). Un correo que Atajos no envió en su momento no llega después: esa compra se agrega con **Pegar un aviso**. Si está en la lista, la app dice por qué no lo registró: era una compra con cargo a la cuenta, en dólares, con otra tarjeta, una compra que eliminaste o un correo que no reconoce como aviso de compra.
4. Mientras tanto, la compra se puede agregar con **Pegar un aviso**.

El botón **Copiar este detalle** copia la lista con el texto de cada correo, para revisar un aviso que la app no reconoce.

**Leer el buzón ahora**, en esa misma hoja, repasa el buzón completo y no solo los correos nuevos. Cuando la app se actualiza con un lector de avisos nuevo, también lo repasa completo una vez al abrirse, para recuperar compras que el lector anterior no reconoció.

## Instalar en el iPhone

1. En GitHub, ve a **Settings → Pages** y publica la rama `main` desde la raíz (`/`). La app queda en `https://pablocarvallo.github.io/master/`.
2. Abre esa dirección en **Safari** en el iPhone.
3. Toca **Compartir → Añadir a pantalla de inicio**.

Se abre como app independiente, con su icono, y funciona sin conexión.

## Archivos

| Archivo | Uso |
| --- | --- |
| `index.html` | La app completa (HTML, CSS y JavaScript sin dependencias de compilación) |
| `manifest.webmanifest` | Nombre, colores e iconos de la app instalada |
| `sw.js` | Service worker para uso sin conexión |
| `icons/icon-full.svg` | Icono original a pantalla completa (fuente de los PNG) |
| `icons/icon.svg` | Icono con esquinas redondeadas (favicon) |
| `icons/apple-touch-icon.png` | Icono de 180 px para la pantalla de inicio del iPhone |
| `icons/icon-192.png`, `icons/icon-512.png`, `icons/icon-1024.png` | Iconos del manifiesto y archivo de alta resolución |

## Datos

Los movimientos y la deuda quedan en el `localStorage` del navegador, en el dispositivo. Este repositorio solo contiene el código de la app. Los correos del banco pasan por el buzón, que es un repositorio privado aparte.

- `master-movimientos`: cada movimiento con tipo (`c` compra, `g` cargo, `a` abono), fecha (`date`), hora (`time`), descripción (`desc`), monto en pesos (`amount`), origen (`mail` o `manual`), tarjeta (`who`: `T` titular o `A` adicional; `card`: los cuatro dígitos del aviso), número de cuotas (`n`), mes en que se paga la cuota 1 (`first`, como `aaaa-mm`) y, en las compras leídas de un aviso, la clave que evita duplicados (`key`).
- `master-tarjeta`: la deuda inicial (`debt`), desde cuándo rige (`at`), el cupo (`limit`) y los avisos eliminados a propósito (`ignored`).
- `master-preferencias`: orden del listado, repositorio y token del buzón, y momento de la última lectura.

## Icono

El icono es un diseño propio: dos tarjetas superpuestas, la del titular y la adicional, en naranja sobre grafito. No reproduce logotipos de Mastercard ni del banco, que pertenecen a sus dueños.
