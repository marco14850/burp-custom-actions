# BurpSuite - Acciones Personalizadas (Custom Actions)

Este repositorio contiene un recopilado de las Custom Actions de BurpSuite que he desarrollado para hacer más eficiente ataques de suplantación de tokens de sesión, uso anónimo, uso de peticiones no documentadas y verificación de archivos comunes.

## Descripción de Scripts 

- **Common-Files.bambda**: Verifica la existencia de archivos comunes como `robots.txt` dentro del dominio de la petición actual.
- **Relevant-Cookies.bambda**: Reduce cookies de forma secuencial, comprueba estado, contenido y cabeceras de respuesta y valida la petición final antes de enviarla a Repeater.
- **Try-Another-Token.bambda**: Reenvía la petición actual con el token de sesión de otro usuario, modificando únicamente la cabecera o cookie especificada. Útil para probar suplantación de identidad.
- **Selected-String-Analyzer.bambda**: Comprueba Base64 mediante decodificación, datos binarios y Luhn. Superar Luhn no demuestra que una tarjeta exista. Para hashes utiliza Hash Identifier.
- **DeleteAuth.bambda**: Envía la petición sin el token ni cookies, para comprobar si la solicitud puede ejecutarse sin autenticación.
- ⚠️ **GET-POST-PUT-PATCH-DELETE.bambda**: Ejecuta la solicitud con distintos métodos HTTP para descubrir funcionalidades expuestas.  
  ⚠️ **Advertencia**: El uso del método DELETE podría causar cambios irreversibles.
- **OPTIONS-TRACE-HEAD-CONNECT.bambda**: Prueba métodos HTTP menos comunes para descubrir superficies adicionales de ataque.
- **Relevant-Headers.bambda**: Reduce cabeceras y cookies secuencialmente, protege cabeceras sin distinguir mayúsculas y valida el resultado en Repeater.
- **HashIdentifier.bambda**: Reconoce formatos compatibles con MD5, SHA-1, SHA-256, SHA-512 y bcrypt; el formato no confirma el algoritmo.
- **Request-Smuggling-Tester.bambda**: Registra estado, tamaño y duración de cuatro sondas CL.TE, TE.CL, H2.CL y H2.TE; informa errores como inconclusos y requiere confirmación manual.
- **Web-Cache-Deception.bambda**: Busca candidatos comparando contenido autenticado y anónimo en rutas únicas, excluyendo respuestas no almacenables. No confirma automáticamente una vulnerabilidad.

## Validación y límites

Las acciones Relevant requieren dos respuestas base estables. Puedes configurar `volatilePattern` para campos dinámicos conocidos; usa patrones específicos para no ocultar diferencias relevantes. Las eliminaciones son acumulativas y la petición final se valida de nuevo; no se garantiza el mínimo global. Conservan el cuerpo original sin convertirlo a texto.

Web Cache Deception requiere una petición GET y conserva su query. Los controles anónimos eliminan únicamente `Authorization` y `Cookie`: elimina otras credenciales y desactiva reglas de sesión que las reintroduzcan. Una coincidencia requiere revisión manual para distinguir caché de fallos de autorización. Las sondas de smuggling no confirman ni descartan vulnerabilidades y deben verificarse en un entorno de pruebas aislado.

Estas acciones reenvían peticiones: Relevant puede repetir operaciones con efectos secundarios. Los errores de transporte no equivalen a vulnerabilidades. La compatibilidad final debe verificarse importando las acciones en tu versión de Burp.

## Instalación

1. Clona el repositorio de github.
```bash
   git clone https://github.com/marco14850/burp-custom-actions.git
```

2. Abre BurpSuite y ve a: Extensions > Bambda Library.
<p align="center">
  <img src="resources/Install-Guide-1.png" alt="Localización de Bambda Library" width="500"/>
</p>

3. Haz clic en el botón Import y selecciona todos los archivos en la carpeta /src.
<p align="center">
  <img src="resources/Install-Guide-2.png" alt="Importar custom actions" width="500"/>
</p>

4. Luego en la pestaña Repeater, ve a Custom Actions > Load y selecciona los archivos de acciones personalizadas.
<p align="center">
  <img src="resources/Install-Guide-3.png" alt="Localización de Custom Actions dentro de Repeater" width="500"/>
</p>
<p align="center">
  <img src="resources/Install-Guide-4.png" alt="Importar custom actions" width="500"/>
</p>

5. Ejecuta las acciones personalizadas desde cualquier solicitud cargada en Repeater.
<p align="center">
  <img src="resources/Install-Guide-5.png" alt="Ejecutar Custom Actions" width="500"/>
</p>

6. Si necesitas personalizar valores (como en el script 'Try Another Token'), edita el script directamente desde BurpSuite y reemplaza el token o cookie deseado.
<p align="center">
  <img src="resources/Install-Guide-6.png" alt="Modificar scripts de Custom Actions" width="500"/>
</p>