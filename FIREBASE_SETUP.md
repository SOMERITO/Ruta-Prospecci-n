# Ruta Prospección — configuración de Firebase

Esta rama contiene una propuesta de inicio de sesión, permisos y sincronización. No la fusiones ni la pongas en producción hasta completar y probar los pasos de configuración.

## 1. Autenticación

En Firebase Console → Authentication → Método de acceso:
- Mantén habilitado **Correo electrónico/contraseña**.
- No habilites el acceso anónimo ni agregues un formulario de registro público.
- Cada persona usará su correo `@uwiener.edu.pe` y una contraseña exclusiva de Ruta Prospección; nunca se solicitará ni almacenará la contraseña de Outlook.

Después de desplegar el HTML, añade a Authentication → Configuración → Dominios autorizados el dominio exacto de GitHub Pages que sirve la página.

## 2. Publicar reglas de Firestore

En Firebase Console → Firestore Database → **Reglas**, pega el contenido completo de `firestore.rules` de esta rama y pulsa **Publicar**. No uses reglas abiertas ni el modo de prueba.

Las reglas:
- Requieren inicio de sesión, correo institucional verificado y un perfil autorizado.
- Impiden que un usuario cree o modifique su propio perfil o su rol.
- Permiten que los asesores consulten y modifiquen solo los registros diarios que les pertenecen.
- Permiten a los asesores consultar sus indicadores y los de todo el equipo, sin editarlos.
- Solo la supervisora puede crear, modificar o eliminar los documentos de métricas mensuales.
- Permiten a personas autorizadas leer la configuración de recursos; solo la supervisora puede modificarla.
- Deniegan cualquier otra ruta.

## 3. Dar de alta cuentas y perfiles

Para cada persona autorizada:
1. En Authentication → Usuarios, elige **Agregar usuario**. Registra su correo institucional y asigna una contraseña temporal aleatoria.
2. Tras el despliegue, la persona podrá pulsar **Olvidé mi contraseña** para recibir un enlace y definir su contraseña propia.
3. En el primer inicio de sesión, la página solicita verificar el correo institucional. La persona debe abrir el enlace recibido y volver a iniciar sesión.
4. Copia el UID que aparece en Authentication → Usuarios.
5. En Firestore → Datos, crea un documento en la colección `authorizedUsers`, usando ese UID exacto como ID del documento.

Campos del perfil:
- `email`: string, correo institucional exacto de esa cuenta.
- `role`: string, `supervisor` para Camila, `asesor` para cuentas individuales o `lector` para la cuenta compartida de consulta.
- `asesorNombre`: string obligatorio solo para perfiles con rol `asesor`.
Las metas no se guardan en el perfil permanente del asesor porque cambian por mes; se registran en el documento mensual correspondiente.

No compartas contraseñas ni las guardes en GitHub. El correo debe verificarse antes de que Firestore permita consultar datos.

## 4. Cuenta compartida de consulta

Para usar una sola cuenta compartida para consultar indicadores, crea un usuario en Firebase Authentication con un correo real, verificable y cuyo buzón de recuperación controle Camila. El HTML de esta rama actualmente acepta correos terminados en `@uwiener.edu.pe`; si el correo general no tiene ese dominio, detente antes de usarlo, porque habrá que ajustar la validación.

Luego crea `authorizedUsers/{UID}` con `email` (string con el correo exacto) y `role` (string: `lector`). No agregues `asesorNombre`. Esa cuenta mostrará solo la pestaña Avance y no podrá editar métricas ni marcar las tareas del registro diario. Como los usuarios comparten credenciales, Firebase no podrá identificar cuál asesor consulta la tabla. Tras la salida de alguien del equipo, Camila puede solicitar el restablecimiento de la contraseña con el correo y distribuir la nueva al resto del grupo autorizado.

La supervisora debe mantener una cuenta separada con rol `supervisor`. Nunca compartas sus credenciales.

## 5. Recursos privados de la ruta

Las URL operativas se cargan desde Firestore, no desde el HTML público.

En Firestore → Datos, crea la colección `appConfig` y el documento `roster` con un campo `advisors` de tipo array que contenga los nombres vigentes del equipo. La supervisora puede gestionar este listado. Después crea el documento `resources`. Añade un campo de tipo **mapa** llamado `links`. Dentro del mapa agrega campos de tipo **string** con estas claves:

- `intranet`
- `grupoRutasUbicaciones`
- `grupoInformeActividades`
- `crmZoho`
- `taxiDirecto`
- `cabify`
- `formularioMaterial`
- `matrizSolicitudes`
- `registroFichasDatos`
- `digitalizacionFichas`
- `actualizacionFichas`
- `statusCartera`

Para cada campo, copia la URL vigente desde la fuente autorizada y confirma que la supervisora permite su uso. No pongas esas URL en `index.html`, en este archivo ni en otros archivos públicos.

## 6. Migración y métricas iniciales

- Los registros diarios se guardan en `trackerRecords`, con un documento por UID y fecha.
- Las métricas se guardan en `monthlyMetrics/{YYYY-MM}`, por ejemplo `monthlyMetrics/2026-10`. Cada mes queda separado; editar noviembre no modifica octubre.
- Cada documento mensual contiene `month` (string), `rows` (array de mapas con `asesor`, `meta`, `avance`, `proyeccion`), `updatedBy` (UID) y `updatedAt` (timestamp).
- Al abrir el mes actual, la aplicación intenta migrar métricas antiguas desde `avanceMetricsV6` del navegador de la supervisora si existe una copia local. Si no existe, los perfiles de asesores se muestran con valores iniciales en cero y la supervisora debe introducir las cifras del mes. No se copian valores a meses anteriores o futuros.
- Cada asesor puede migrar desde su propio navegador los registros locales de su nombre, siempre que esa fecha todavía no esté en Firestore.
- Los valores de métricas mostrados como datos iniciales en la versión anterior del HTML no se deben considerar migrados automáticamente. Antes de producción, comprueba los valores de origen y cárgalos en Firestore si siguen siendo vigentes.

## 7. Aviso sobre el historial público

La rama actual deja de incrustar nombres del equipo, métricas iniciales y URL operativas en el HTML. Sin embargo, el repositorio principal sigue siendo público y los commits anteriores pueden conservar información que ya se publicó. Fusionar este cambio no borra el historial.

Antes de usar datos reales:
- Revisa si corresponde hacer privado el repositorio o mover el despliegue a un alojamiento apropiado; comprueba cómo afectaría a la página actual.
- Si una invitación a un grupo privado de WhatsApp estuvo expuesta y no debía ser pública, solicita que se regenere.
- No incluyas datos reales de prospectos/alumnos hasta completar la revisión de privacidad y seguridad.

## 8. Verificaciones antes de producción

- Probar con la cuenta supervisora de Camila y la cuenta compartida `lector`.
- Abrir la sección Avance con una cuenta de asesor y confirmar que puede ver toda la tabla, pero no tiene botón para editar.
- Cambiar de mes en la sección Avance como supervisora, editar una meta y confirmar que solo cambia el documento de ese mes.
- Comprobar que un correo institucional no autorizado no accede a Firestore.
- Confirmar que un asesor no puede consultar los registros de otro ni modificar métricas globales.
- Probar la sincronización en dos navegadores o dispositivos.
- Verificar recuperación de contraseña, confirmación del correo y cierre de sesión.
- Mantener el proyecto en Spark sin vincular facturación y revisar el uso de cuotas.
