# Master

App personal para llevar la tarjeta de crédito Mastercard: lee los avisos de compra que el banco envía por correo, guarda fecha, comercio, monto y tarjeta (Titular o Adicional), y muestra el cupo disponible y el utilizado.

Es un registro personal. No es la app oficial del banco ni está conectada a la tarjeta.

## Qué hace

- **Disponible y utilizado**: se parte escribiendo el cupo y el utilizado que muestra hoy la app del banco. Desde ahí, cada compra o cargo resta del disponible y cada abono suma. Si en algún momento no coincide con el banco, **Ajustar** lo corrige escribiendo el utilizado del banco.
- **Lectura automática**: una automatización de Atajos envía cada correo del banco, apenas llega a Mail, a un buzón privado en GitHub. La app lee ese buzón al abrirse y agrega solo las compras con tarjeta de crédito que falten. Un mismo aviso nunca se registra dos veces, y si la compra ya estaba anotada a mano (mismo día y mismo monto), el aviso se une a ella en vez de duplicarla.
- **Pegar un aviso**: también se puede copiar el texto de un correo del banco y pegarlo.
- **Titular o Adicional**: el aviso trae los últimos cuatro dígitos de la tarjeta. `****9908` se registra como Titular y `****4526` como Adicional.
- **Compra, cargo y abono a mano**: sirve para los cobros de la tarjeta (comisiones, intereses, seguros) y para los pagos o devoluciones. La hora es opcional.
- **Cuotas pendientes**: una compra en cuotas se anota con el monto total y el número de cuotas. Las cuotas que quedan de compras anteriores se agregan en la sección **Cuotas pendientes** con lo que falta por cobrar. La sección muestra cuánto queda por cobrar y en qué facturación se cobra cada cuota.
- **Totales por tarjeta**: cuánto suman las compras del Titular y del Adicional, más los cargos y los abonos.
- **Extracto**: el detalle de todo lo registrado, del saldo anterior al utilizado. Se copia como texto, se comparte o se exporta a Excel.
- **Borrar registros anteriores a una fecha**: por ejemplo, los de un extracto ya pagado. Su monto pasa al saldo anterior, así el utilizado y el disponible no cambian.
- **Borrar un registro**: al tocar un movimiento se corrige o se elimina. Hay unos segundos para deshacer.
- **Widget en la pantalla de inicio**: muestra el disponible, el utilizado y las tres últimas compras, sin abrir la app.
- **Datos y respaldo**: copia todos los datos como texto y los restaura desde ahí.

## Cómo se calcula

- **Utilizado** = saldo anterior + compras + cargos + cuotas por cobrar de compras anteriores − abonos.
- **Disponible** = cupo − utilizado.
- El **saldo anterior** es lo utilizado antes del primer día que registra la app, sin las cuotas que se anotan aparte en «Cuotas pendientes». La app lo calcula sola: al partir y en **Ajustar** se escribe el utilizado del banco y el saldo anterior queda en lo que haga falta para que coincidan.
- Las compras de días anteriores al saldo ya están dentro de él: los avisos de esos días no se registran y no se pueden anotar movimientos con esas fechas.
- Una compra en cuotas ocupa el cupo por su monto total desde el día de la compra, igual que en el banco, y se libera con cada abono.

## Cuotas

- Cada cuota se cobra en la **facturación** de un mes, una por mes. Al anotar una compra en cuotas se indica en qué facturación se cobra la cuota 1. La app sugiere la del mismo mes si la compra es hasta el día 22 y la del mes siguiente si es posterior (el banco cierra la facturación cerca de ese día; se cambia en `CIERRE`, al comienzo del código).
- Una cuota queda **cobrada** cuando cierra su facturación. Hasta entonces está **por cobrar**.
- Lo que falta por cobrar ya está dentro del utilizado. Las cuotas no se marcan a mano y no cambian el saldo: el disponible sube cuando se anota el abono.
- El valor de cada cuota es el total dividido en partes iguales, sin intereses; la última absorbe el redondeo.
- **Cuotas por cobrar de una compra anterior**: se agregan en «Cuotas pendientes» con el comercio, el monto que falta, cuántas cuotas quedan y en qué facturación se cobra la próxima.

## Borrar registros anteriores a una fecha

Al final de **Movimientos**, **Borrar registros anteriores a una fecha** borra los movimientos de los días anteriores a la fecha elegida:

- Su monto pasa al saldo anterior, que queda a esa fecha. El utilizado y el disponible no cambian.
- Las compras en cuotas que aún tienen cuotas por cobrar no se pierden: lo que falta por cobrar queda en «Cuotas pendientes».
- Antes de borrar muestra cuántos registros se van, el saldo anterior que resulta y pide tocar dos veces. Después hay unos segundos para deshacer.

## Extracto

**Generar extracto** muestra todo lo registrado desde el saldo anterior:

- Cupo, utilizado y disponible.
- El saldo anterior y cada movimiento, con los montos como deuda: compras, cargos y cuotas en positivo, abonos en negativo.
- **Totales**: saldo anterior + compras del Titular + compras del Adicional + cargos − abonos + cuotas de compras anteriores = utilizado.
- **Cuotas pendientes**: lo que falta por cobrar de cada compra en cuotas y en qué facturaciones.

### Exportar a Excel

**Exportar a Excel** genera el archivo `Extracto Master aaaa-mm-dd.xlsx` con estas hojas:

- **Resumen**: cupo, utilizado y disponible, más el detalle que lleva del saldo anterior al utilizado.
- **Movimientos**: el saldo anterior y una fila por movimiento, con número, fecha, hora, descripción, tipo, tarjeta, cuotas y monto. La suma de la columna «Monto» es el utilizado. Trae filtros y la primera fila fija.
- **Cuotas**: una fila por compra en cuotas, con el valor de la cuota, la primera y la última facturación, las cobradas, las que faltan y la próxima facturación. Solo aparece si hay cuotas.

En el iPhone el archivo se entrega por la hoja de compartir, para guardarlo en Archivos, abrirlo en Excel o Numbers, o enviarlo. En un computador se descarga. El archivo se arma en el dispositivo, sin conexión y sin enviar los datos a ningún servicio.

## Formato del aviso

El lector reconoce el aviso de compra con tarjeta de crédito del Banco de Chile (remitente `enviodigital@bancochile.cl`):

> Te informamos que se ha realizado una compra por $7.930 con Tarjeta de Crédito ****4526 en RedGloba*SU CASERO el 30/07/2026 09:34. Revisa Saldos y Movimientos en App Mi Banco o Banco en Línea.

De cada aviso toma el monto, la tarjeta, el comercio, la fecha y la hora. Funciona con el texto plano o con el correo en HTML, y también si el correo llega codificado (con `=C3=A9` y líneas cortadas con `=`, o en base64).

La compra se reconoce como de la tarjeta por su **terminación** (`9908` o `4526`), no por la palabra «Crédito»: según cómo Atajos lea el correo, esa palabra puede llegar con el acento dañado (`CrÃ©dito`, `Cr?dito`).

Antes de leer el aviso, el lector repara el texto mal decodificado (`Ã©` vuelve a ser `é`, y `Â` seguido de un espacio duro vuelve a ser un espacio) y tolera signos sueltos entre las palabras. Un mismo aviso no se registra dos veces aunque llegue una vez con el texto dañado y otra bien, porque las compras se comparan sin signos ni acentos en el comercio.

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
- Las compras del buzón anteriores al saldo anterior no se registran: ya están dentro de él.
- Si una compra ya está anotada a mano con la misma fecha y el mismo monto, su aviso se une a ella y no se duplica. Así se pueden anotar compras a mano (por ejemplo, desde la cartola del banco) y dejar que los avisos lleguen después.
- La lectura del buzón funciona en la app publicada en GitHub Pages. Dentro de Claude, la página no puede llamar a GitHub y las compras se agregan pegando el aviso.

### Si llega un correo y la compra no aparece

1. Toca **Actualizar**. Al volver a la app, el buzón se relee solo si pasaron más de dos minutos desde la última lectura.
2. Abre **Buzón y atajo** y mira **Últimos correos del buzón**. Ahí están los últimos correos que recibió el buzón, del más reciente al más antiguo, con lo que la app hizo con cada uno.
3. Si el correo no está en la lista, no llegó al buzón: el problema está en la automatización de Atajos (tiene una condición de asunto que lo dejó fuera, no se ejecutó o GitHub rechazó el envío). Un correo que Atajos no envió en su momento no llega después: esa compra se agrega con **Pegar un aviso**. Si está en la lista, la app dice por qué no lo registró: era una compra con cargo a la cuenta, en dólares, con otra tarjeta, anterior al saldo anterior, una compra que eliminaste o un correo que no reconoce como aviso de compra.
4. Mientras tanto, la compra se puede agregar con **Pegar un aviso**.

El botón **Copiar este detalle** copia la lista con el texto de cada correo, para revisar un aviso que la app no reconoce.

**Leer el buzón ahora**, en esa misma hoja, repasa el buzón completo y no solo los correos nuevos. Cuando la app se actualiza con un lector de avisos nuevo, también lo repasa completo una vez al abrirse, para recuperar compras que el lector anterior no reconoció.

## Widget en la pantalla de inicio

Muestra el disponible, el utilizado y las tres últimas compras, cada una con una **T** (Titular) o una **A** (Adicional). Una app web no puede crear widgets en iPhone, así que el widget se hace con **Scriptable**, una app gratuita que dibuja widgets a partir de un script.

1. Instala **Scriptable** desde el App Store.
2. En Master, toca **Widget**, bajo los botones de arriba (o entra a **Buzón y atajo → Widget en la pantalla de inicio**), y toca **Copiar el script del widget**. El script va completo, con el buzón y el token. Si el buzón no está conectado, la misma hoja pide conectarlo primero. Dentro de Claude no hay buzón ni widget: se copia desde la app instalada en el iPhone.
3. Abre Scriptable, toca **+** y pega el script. Nómbralo **Master** y ejecútalo con **▶**: debe mostrar el disponible.
4. En la pantalla de inicio, mantén presionado un espacio vacío, toca **Editar → Agregar widget**, busca **Scriptable** y elige el tamaño mediano.
5. Mantén presionado el widget nuevo, toca **Editar widget** y en **Script** elige **Master**.

Cómo funciona:

- Al activarlo, Master publica el utilizado, el cupo y las últimas compras en un hito (*milestone*) del repositorio privado del buzón, llamado «Master · widget». El script los lee y les suma las compras con la tarjeta que llegaron al buzón después, así el widget se pone al día aunque no abras Master.
- Si el script se copió antes de esta versión (cuando mostraba el total a pagar), hay que copiarlo de nuevo desde Master y reemplazarlo en Scriptable.
- iOS decide cada cuánto actualiza el widget, normalmente cada varios minutos. No es instantáneo.
- Los cargos, los abonos, los ajustes y las compras anotadas a mano solo existen dentro de Master: el widget los refleja después de abrir la app.
- Al tocar el widget se abre Scriptable, no Master: iOS no permite abrir una app web instalada desde un widget.
- También sirve en tamaño pequeño y en la pantalla bloqueada (rectangular).
- El widget de Master y el de FAN conviven: usan el mismo buzón, pero cada uno tiene su propio script y su propio hito.
- **Desactivar el widget**, en la misma sección de Master, deja de publicar y borra lo publicado.

## Instalar en el iPhone

1. En GitHub, ve a **Settings → Pages** y publica la rama `main` desde la raíz (`/`). La app queda en `https://pablocarvallo.github.io/master/`.
2. Abre esa dirección en **Safari** en el iPhone.
3. Toca **Compartir → Añadir a pantalla de inicio**.

Se abre como app independiente, con su icono, y funciona sin conexión. No se agranda con los dedos ni con doble toque.

## Archivos

| Archivo | Uso |
| --- | --- |
| `index.html` | La app completa (HTML, CSS y JavaScript sin dependencias de compilación) |
| `manifest.webmanifest` | Nombre, colores e iconos de la app instalada |
| `sw.js` | Service worker para uso sin conexión |
| `widget/master-widget.js` | Script del widget para la app Scriptable. Master lo entrega completo con el botón de copiar; `index.html` lleva una copia idéntica entre las marcas `WIDGET:START` y `WIDGET:END`, que hay que renovar si el script cambia |
| `icons/icon-full.svg` | Icono original a pantalla completa (fuente de los PNG) |
| `icons/icon.svg` | Icono con esquinas redondeadas (favicon) |
| `icons/apple-touch-icon.png` | Icono de 180 px para la pantalla de inicio del iPhone |
| `icons/icon-192.png`, `icons/icon-512.png`, `icons/icon-1024.png` | Iconos del manifiesto y archivo de alta resolución |

## Datos

Los movimientos y el saldo quedan en el `localStorage` del navegador, en el dispositivo. Este repositorio solo contiene el código de la app. Los correos del banco pasan por el buzón, que es un repositorio privado aparte.

- `master-movimientos`: cada movimiento con tipo (`c` compra, `g` cargo, `a` abono, `q` cuotas por cobrar de una compra anterior), fecha (`date`), hora opcional (`time`), descripción (`desc`), monto en pesos (`amount`), origen (`mail` o `manual`), tarjeta (`who`: `T` titular o `A` adicional; `card`: los cuatro dígitos del aviso), número de cuotas (`n`), facturación de la primera cuota (`prox`, como `aaaa-mm`) y, en las compras leídas de un aviso, la clave que evita duplicados (`key`).
- `master-tarjeta`: el cupo (`limit`), el primer día que registra la app (`desde`), el saldo anterior a ese día (`saldo`) y los avisos eliminados a propósito (`ignored`).

Los datos y los respaldos de la versión anterior (deuda inicial con fecha y hora, periodos con fecha de corte) se leen solos: la deuda inicial pasa a ser el saldo anterior.
- `master-preferencias`: orden del listado, repositorio y token del buzón, momento de la última lectura y si el widget está activado.

## Icono

El icono es un diseño propio: dos tarjetas superpuestas, la del titular y la adicional, en naranja sobre grafito. No reproduce logotipos de Mastercard ni del banco, que pertenecen a sus dueños.
