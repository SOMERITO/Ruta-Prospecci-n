# Ruta Prospección — configuración de Firebase

Esta rama contiene una propuesta de inicio de sesión, permisos y sincronización. No la fusiones ni la pongas en producción hasta completar y probar los pasos de configuración.

## 1. Autenticación

En Firebase Console → Authentication → Método de acceso:
- Mantén habilitado **Correo electrónico/contraseña**.
- No habilites el acceso anónimo ni agregues un formulario de registro público.
- Camila ingresa con su cuenta institucional autorizada.
- El equipo utiliza la única cuenta compartida exacta `prospeccionwiener@gmail.com`, con una contraseña exclusiva de Ruta Prospección. La contraseña de Gmail no se comparte con los asesores.
- La cuenta general debe verificarse una vez desde el enlace enviado a ese buzón, que controla Camila.

Después de desplegar el HTML, añade a Authentication → Configuración → Dominios autorizados el dominio exacto de GitHub Pages que sirve la página.

## 2. Publicar reglas de Firestore

En Firebase Console → Firestore Database → **Reglas**, pega el contenido completo de `firestore.rules` de esta rama y pulsa **Publicar**. No uses reglas abiertas ni el modo de prueba.

Las reglas:
- Requieren inicio de sesión, correo verificado (institucional para Camila y exacto `prospeccionwiener@gmail.com` para la cuenta compartida) y un perfil autorizado.
- Impiden que un usuario cree o modifique su propio perfil o su rol.
- Una cuenta compartida con rol `asesor_compartido` puede guardar el avance diario bajo el nombre seleccionado; una cuenta individual con rol `asesor`, si se usa, solo puede cambiar sus propios registros.
- Todos los perfiles autorizados pueden consultar los indicadores mensuales del equipo.
- Solo la supervisora puede crear, modificar o eliminar los documentos de métricas mensuales.
- Permiten a personas autorizadas leer la configuración de recursos; solo la supervisora puede modificarla.
- Deniegan cualquier otra ruta.

## 3. Dar de alta cuentas y perfiles

Para crear una cuenta autorizada:
1. En Authentication → Usuarios, elige **Agregar usuario**.
2. Para Camila, usa su correo institucional verificado. Para la cuenta del equipo, registra exactamente `prospeccionwiener@gmail.com` y define una contraseña aleatoria exclusiva de la aplicación.
3. Copia el UID que aparece en Authentication → Usuarios.
4. En Firestore → Datos, crea un documento en la colección `authorizedUsers`, usando ese UID exacto como ID del documento.
5. Verifica el correo de la cuenta general desde el enlace que llegue a Gmail. Camila puede restablecer la contraseña de Ruta Prospección desde ese mismo buzón; no debe compartir la contraseña de Gmail.

Campos del perfil:
- `email`: string, correo exacto de la cuenta; Camila usa el institucional y la cuenta compartida usa `prospeccionwiener@gmail.com`.
- `role`: string, `supervisor` para Camila, `asesor_compartido` para la cuenta general del equipo o `asesor` para cuentas individuales opcionales.
- `asesorNombre`: obligatorio solo para perfiles con rol `asesor`.
Las metas no se guardan en el perfil permanente del asesor porque cambian por mes; se registran en el documento mensual correspondiente.

No compartas contraseñas ni las guardes en GitHub. El correo debe verificarse antes de que Firestore permita consultar datos.

## 4. Cuenta compartida para registrar progreso

En Authentication → Usuarios, crea un solo usuario con el correo exacto `prospeccionwiener@gmail.com` y una contraseña exclusiva de Ruta Prospección. No uses ni compartas la contraseña de Gmail. La dirección exacta está autorizada en el código y en las reglas de esta rama; no se autoriza cualquier cuenta de Gmail.

Después crea `authorizedUsers/{UID}` con:
- `email`: string, exactamente `prospeccionwiener@gmail.com`.
- `role`: string con el valor `asesor_compartido`.

No agregues `asesorNombre` al perfil compartido. Al entrar, cada persona selecciona su nombre en la pestaña **Asesor**; sus casillas se guardan en un documento distinto por asesor y fecha, y los cambios se sincronizan en tiempo real con el panel de Camila. La cuenta compartida puede consultar todos los indicadores en Avance, pero no editar las métricas. Camila conserva su cuenta separada con rol `supervisor` y es la única que modifica metas mensuales.

En el primer acceso, verifica `prospeccionwiener@gmail.com` desde el enlace que Firebase envía a ese buzón. Camila controla el buzón y la recuperación; los asesores reciben únicamente la contraseña de la aplicación. Si alguien deja el equipo, restablece la contraseña de Ruta Prospección y comparte la nueva solo con quienes sigan autorizados.

**Limitación del acceso compartido:** Firebase no puede verificar qué persona física usa el nombre seleccionado. Cualquiera que conozca la contraseña compartida puede seleccionar otro asesor y modificar su progreso. La actividad queda atribuida a la cuenta general, no al asesor individual. Además, el permiso técnico de la cuenta compartida permite leer registros diarios de todo el equipo para que funcione la sincronización. Si necesitan impedir que una persona pueda ver o modificar registros ajenos, deberán usar cuentas individuales u otro método de autenticación por persona.

## 5. Lista de asesores y recursos privados

La aplicación ya incluye dos paneles exclusivos para la cuenta con rol `supervisor`, dentro de **Dashboard Jefatura**: **Administrar lista de asesores** y **Administrar enlaces de trabajo**. Los cambios se guardan en Firestore, y las reglas permiten editarlos únicamente a la supervisora.

Si hay que crear o reconstruir documentos inicialmente, en Firestore → Datos:
- `appConfig/roster`: mapa `advisorMap`, donde cada clave es el nombre normalizado (minúsculas, sin tildes, palabras separadas por guiones bajos) y cada valor es el nombre completo que debe mostrar la página.
- `appConfig/resources`: mapa `links`, con URLs aprobadas para el equipo.

Claves activas para los enlaces:
- `intranet`, `grupoRutasUbicaciones`, `grupoInformeActividades`, `crmZoho`
- `avanza`, `plantillaVan`, `driveMovilidades`
- `matrizSolicitudes`, `registroFichasDatos`, `digitalizacionFichas`, `actualizacionFichas`, `statusCartera`

Usa enlaces HTTPS y copia las URL desde las fuentes autorizadas. En el panel de enlaces, cada fila tiene el formato `clave=URL`; las filas vacías se ignoran. Al guardar, la aplicación elimina las claves antiguas `taxiDirecto`, `cabify` y `formularioMaterial`. Deben confirmarse los enlaces estables de cartera y Drive, además de los enlaces vigentes de los grupos de WhatsApp, antes de producción.

No pongas nombres del equipo ni URL operativas directamente en el HTML, este archivo o cualquier otro archivo público. No registres contraseñas, códigos de acceso ni enlaces que contengan tokens temporales.

## 6. Migración e historial mensual

- Los registros diarios se guardan en `trackerRecords`. Con la cuenta compartida, cada documento usa la clave `shared_{advisorKey}_{YYYY-MM-DD}`, para separar el progreso por asesor y fecha. La vista de supervisión se actualiza mediante el listener en tiempo real de Firestore.
- Las métricas se guardan en `monthlyMetrics/{YYYY-MM}`, por ejemplo `monthlyMetrics/2026-10`. Cada mes queda separado; editar otro mes no debe cambiar los datos de un mes histórico.
- Cada documento mensual contiene `month`, `rows` (array con `asesor`, `meta`, `avance`, `referidos`, `proyeccion`), `mobilityRows` (array con `asesor`, `asignado`, `gasto`, cuando exista información de movilidades), `updatedBy` y `updatedAt`.
- La columna Meta se interpreta como meta de **4.º de secundaria**. Los porcentajes, la proyección y el logro final mantienen las fórmulas existentes; referidos se suma por separado.
- Al abrir el mes actual, la aplicación intenta migrar métricas desde `avanceMetricsV6` del navegador de la supervisora si existe una copia local. Si no existe, la supervisora debe importar o introducir las cifras reales del mes.
- Los valores iniciales que solo aparecían en la versión anterior del HTML no se consideran migrados automáticamente. Los paneles **Importar historial de Avance** y **Importar o reconstruir un mes desde una tabla** permiten a Camila cargar datos históricos sin incrustar nombres o valores en el HTML público. Antes de importar, selecciona el mes correcto.
- Para importar métricas, pega una fila por asesor con el formato `nombre;meta;avance;referidos;proyección`; también acepta `nombre;meta;avance;proyección`. Las filas se combinan por nombre para mantener otros registros.
- Para importar movilidades, pega una fila por asesor con el formato `nombre;asignado;gasto`. La aplicación calcula saldo y porcentaje de ejecución. El reporte de septiembre debe cargarse explícitamente desde su propia tabla histórica, sin reasignar sus montos a los nombres actuales.
- El presupuesto predeterminado para nuevos registros de movilidades es S/ 2,500 por asesor en el mes actual. La supervisora debe confirmar que ese criterio sigue siendo el aplicable al período antes de usar los totales como reporte oficial.

## 7. Aviso sobre el historial público

La rama actual deja de incrustar nombres del equipo, métricas iniciales y URL operativas en el HTML. Sin embargo, el repositorio principal sigue siendo público y los commits anteriores pueden conservar información que ya se publicó. Fusionar este cambio no borra el historial.

Antes de usar datos reales:
- Revisa si corresponde hacer privado el repositorio o mover el despliegue a un alojamiento apropiado; comprueba cómo afectaría a la página actual.
- Si una invitación a un grupo privado de WhatsApp estuvo expuesta y no debía ser pública, solicita que se regenere.
- No incluyas datos reales de prospectos/alumnos hasta completar la revisión de privacidad y seguridad.

## 8. Verificaciones antes de producción

- Iniciar sesión con la cuenta supervisora de Camila y con la cuenta compartida autorizada; confirmar que otra cuenta de Gmail no puede acceder como equipo.
- En Dashboard Jefatura, editar la lista y un enlace con rol supervisor, guardar, recargar e inspeccionar desde otro navegador. Comprobar que la cuenta compartida no puede abrir ni guardar cambios en esos paneles.
- Actualizar una casilla diaria desde un navegador y confirmar que el Dashboard de Camila refleja el cambio en tiempo real desde un segundo dispositivo.
- Confirmar que la cuenta compartida ve Avance y Movilidades, pero no puede modificar métricas ni presupuesto.
- Probar Referidos postulantes y sus totales, y verificar que el logro final y los porcentajes existentes siguen calculándose según las fórmulas previas.
- Importar el reporte de movilidades de septiembre en el mes de septiembre y confirmar que los diez registros, el total asignado, el gasto, el saldo y la ejecución coinciden con la fuente original.
- Importar o recuperar las métricas históricas en el mes que corresponda. Confirmar que los asesores actuales no reciben valores de antiguos integrantes y que los registros históricos siguen visibles en su mes original.
- Cambiar entre varios meses; comprobar que editar un mes no altera el documento de otro mes.
- Verificar acceso a los enlaces de cartera, Avanza, VAN, Drive, formularios y grupos con una cuenta autorizada. No publicar el sitio hasta que los enlaces caducados o vacíos hayan sido reemplazados por enlaces estables.
- Verificar recuperación de contraseña, correo confirmado y cierre de sesión.
- Mantener Firebase en Spark sin vincular facturación y revisar el uso de cuotas. La comprobación estática de sintaxis no reemplaza una prueba integral con cuentas y dos dispositivos reales.
