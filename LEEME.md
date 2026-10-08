# Portal de Comparativos de Compra

## Archivos
- `index.html` — la aplicación.
- `config.js` — aquí van la URL y la llave pública de Supabase.
- `supabase_setup.sql` — crea las tablas, la seguridad y los 3 aprobadores.

## Instalación (una sola vez)
1. **Supabase**: usa un proyecto que ya tengas, por ejemplo el de sobrepedidos. No hace falta uno nuevo: todas las tablas y funciones de este portal empiezan con `cmp_` y no tocan las que ya existen.
2. En ese proyecto abre **SQL Editor → New query** y pega `supabase_setup.sql`.
   Antes de ejecutarlo, cambia en la sección 10 los nombres reales del Jefe de Costos y del Gerente de Compras y sus PIN iniciales. Luego presiona **Run**.
3. En **Project Settings → API** de ese mismo proyecto copia la *Project URL* y la llave *anon public* y pégalas en `config.js`.
4. **GitHub**: crea el repositorio (ej. `ftrevinor001-tr/Portal-comparativos`), sube `index.html` y `config.js` y activa **Settings → Pages** (rama main, carpeta raíz).
5. Entra como FERNANDO TREVIÑO → **Administración → Cargar catálogo**. Sube *Productos por Estructura Comercial.xlsx*. Así se cargan las claves y se dan de alta los compradores. Después sube el reporte de precio base / último costo del reporteador, que solo actualiza esas columnas.
6. Cada aprobador cambia su PIN en **Administración → Cambiar mi PIN**.

Si `config.js` está vacío, el portal abre en **modo demostración**, con datos de ejemplo que se guardan solo en ese navegador.

## Formato del comparativo
Las columnas siguen el formato de la Comparativa de precios:
- ID, Descripción, Cantidad, Clasificación, Inversión, Pron, Antec, Crecimiento y Último precio.
- Por cada proveedor: Precio, Dif vs últ. $ (el % de aumento) y, si aplica, Precio + % de flete.
- Al final: Costo mínimo, Dif vs últ. costo, Proveedor asignado, Impacto $ y Motivo.
- Una fila al pie con el tipo de entrega de cada proveedor.

El **último precio** se toma del precio aprobado vigente de la clave. Si todavía no tiene uno, se usa el de la última compra del catálogo. También se puede capturar a mano o importar del archivo.

Al importar un Excel con este formato:
- El % de flete se lee de encabezados como "TYMSA + 3% FLETE".
- El tipo de entrega se toma de la fila de notas al pie.
- Las columnas calculadas (Dif, Costo mínimo, Proveedor) se recalculan solas.

## Cómo funciona
- **Compradores**: entran eligiendo su nombre. Crean comparativos, ya sea capturando o importando su Excel, y los envían a aprobación.
- **Aprobadores** (Jefe de Compras, Jefe de Costos y Gerente de Compras): entran con su PIN. Basta que uno apruebe. Queda registrado quién aprobó, cuándo y su comentario.
- Al aprobar, cada clave toma el **proveedor y precio elegidos como vigentes**. El anterior se conserva en el historial.
- Al capturar una clave en un comparativo nuevo, aparece su proveedor vigente y el último precio aprobado.
- Los proveedores tienen estatus Aprobado, En evaluación o Suspendido. Solo los aprobadores cambian ese estatus, con su PIN.
- Hay un registro de accesos y una bitácora de todos los cambios. No se borra nada.

## Proveedores por clave y bloqueos (actualización 02)
**Instalación:** en Supabase abre **SQL Editor → New query**, pega `supabase_update_02.sql` y presiona **Run**. Después reemplaza `index.html` en GitHub; `config.js` no cambia.

- **Cargar el historial:** entra a **Administración → Proveedores por clave** y sube un Excel con Clave, Proveedor, Tipo (Compra / Cotización), Fecha, Precio y Clave del proveedor. Puedes usar la *Plantilla proveedores por clave.xlsx*.
- **En el comparativo:** al capturar una clave, se agregan solos los proveedores a los que se les ha comprado o cotizado esa clave. Cada celda indica si fue *comprado* o *cotizado*. Si un proveedor con historial se queda sin precio, la celda se marca y el portal lo avisa al enviar a aprobación.
- **Bloquear para una clave:** entra a **Consultar clave → Proveedores de esta clave → Bloquear**.
- **Bloquear para todas las claves:** entra a **Proveedores**, abre el proveedor y presiona **Bloquear en todas las claves**.
- **Ver y quitar bloqueos:** entra a **Administración → Bloqueos**.
- **Qué hace un bloqueo:** el proveedor ya no se agrega solo, su celda queda deshabilitada y no se le puede asignar la clave, ni siquiera al aprobar.
- **Quién bloquea:** solo los aprobadores, con su PIN. Todo queda en la bitácora.
- **Al aprobar un comparativo:** todos los proveedores que dieron precio quedan registrados como *cotizados* para esa clave.

## Archivos "Proveedores por clave" de cada comprador (actualización 03)
**Instalación:** en Supabase ejecuta `supabase_update_03.sql`, igual que la 02. Después reemplaza `index.html` en GitHub.

Para cargar los archivos entra a **Administración → Proveedores por clave** y selecciona uno o varios archivos de comprador a la vez. No hace falta consolidarlos en uno solo. Al cargarlos:
- **Compras:** salen de la hoja *Detalle Clave-Proveedor*, con recepciones, importe, primera y última compra y la clave del producto del proveedor.
- **Cotizados:** salen de las columnas *COTIZADO 1 a 5*.
- **Proveedores:** se identifican por su **ID del sistema**, así no se duplican aunque el nombre cambie. A los proveedores que ya existan en el portal ponles su ID en **Proveedores → (abrir) → ID en el sistema**.
- **Reemplazo:** con la casilla "Reemplazar" activada, solo se reemplaza lo de las claves que vienen en los archivos. Las demás claves y lo que viene de comparativos aprobados se conservan.
- **Observaciones:** al final aparecen las de los compradores, para que decidas si hay que bloquear algo.
