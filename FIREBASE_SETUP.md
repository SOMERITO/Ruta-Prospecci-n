# Ruta Prospección — configuración de Firebase

Esta rama prepara inicio de sesión privado y sincronización en Firestore. **No debe fusionarse ni ponerse en producción hasta publicar las reglas y crear los perfiles autorizados.**

## 1. Proveedor de autenticación

En Firebase Console → Authentication → Método de acceso:
- Mantener habilitado **Correo electrónico/contraseña**.
- No habilitar el registro público desde la interfaz. La página solo ofrece iniciar sesión y restablecer contraseña.

Las cuentas se provisionan manualmente. Cada integrante usa su correo `@uwiener.edu.pe`, pero la contraseña es exclusiva de Ruta Prospección. Nunca se debe pedir ni guardar su contraseña de Outlook.

## 2. Publicar reglas de Firestore

En Firebase Console → Firestore Database → **Reglas**, copiar el contenido completo de `firestore.rules` de este repositorio y pulsar **Publicar**.

Las reglas:
- Exigen una cuenta institucional verificada.
- Permiten leer el propio perfil de `authorizedUsers`, pero impiden que clientes creen o editen perfiles y roles.
- Permiten a asesores leer y escribir solo sus propios documentos de `trackerRecords`.
- Permiten a la supervisora consultar los registros del equipo y administrar `appData/avanceMetrics`.
- Deniegan las rutas no declaradas.

## 3. Crear cuentas autorizadas

Para cada usuario:
1. Authentication → Usuarios → **Agregar usuario**. Registra su correo institucional y genera una contraseña temporal aleatoria que no se reutilice.
2. La persona puede usar **Olvidé mi contraseña** en la página para definir su propia contraseña mediante el correo de restablecimiento.
3. Tras iniciar sesión, la página enviará un mensaje de verificación. Debe verificar el correo y volver a ingresar.
4. En Firestore → Datos, crea una colección `authorizedUsers` y un documento cuyo ID sea el **UID** exacto de Authentication.

Campos del documento:
- `email` — string, exactamente el correo de la cuenta.
- `role` — string; usa `supervisor` o `asesor`.
- `asesorNombre` — string obligatorio para `asesor`. El nombre se carga desde este perfil y ya no se encuentra escrito en el HTML público.
- `meta` — number opcional para asesores; meta del período usada al inicializar métricas nuevas.

Los UID y perfiles se obtienen desde Authentication → Usuarios. No compartas contraseñas ni las guardes en el repositorio.

## 4. Datos compartidos

- Los avances diarios se guardan en `trackerRecords`, con un documento por UID y fecha.
- Las métricas del equipo se guardan en `appData/avanceMetrics`; la supervisora es quien las administra.
- La primera sesión de supervisora inicializa las métricas desde el respaldo local del navegador si existe; de lo contrario, usa los valores base que ya tiene la aplicación.
- La primera sesión de cada asesor intenta migrar los registros locales de ese mismo asesor si la fecha aún no existe en Firestore.

## 8. Verificaciones antes de producción

- Probar con una cuenta supervisora y una cuenta asesora.
- Confirmar que una cuenta no autorizada no puede consultar datos.
- Confirmar que un asesor no puede consultar registros de otro asesor ni modificar métricas globales.
- Confirmar la sincronización en dos navegadores/dispositivos.
- Mantener el proyecto en Spark sin vincular facturación.

## Nota de privacidad pendiente

El repositorio actual es público y el HTML existente contiene nombres, métricas iniciales y enlaces a recursos internos. Una pantalla de inicio de sesión no vuelve privado el código estático. Antes de fusionar o usar con datos reales, se debe revisar y retirar del HTML público la información no autorizada para difusión o definir una estrategia de alojamiento/repositorio privado compatible con el despliegue actual.


## 6. Recursos privados de la ruta

Las URL operativas se leen desde el documento `appConfig/resources` de Firestore y no deben volver a pegarse en `index.html`, porque el repositorio es público.

Crea el documento `appConfig/resources` desde la consola de Firestore con un campo de tipo mapa llamado `links`. Dentro del mapa, agrega estas claves de tipo string y asigna a cada una la URL aprobada del recurso:

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

Las URL no se incluyen en este archivo de instrucciones. Recupéralas desde la fuente autorizada y confirma que son correctas antes de cargarlas. Una vez creado el documento, los miembros autorizados podrán abrir los recursos y el HTML público solo contendrá claves genéricas.

## 7. Aviso sobre el historial público

Quitar datos de la versión actual de `index.html` no borra automáticamente los commits anteriores de un repositorio público. Antes de considerar estos recursos privados, revisa la visibilidad del repositorio y el historial. Si alguno de los enlaces es una invitación a un grupo privado y no debería haber sido público, solicita que se regenere ese enlace.
