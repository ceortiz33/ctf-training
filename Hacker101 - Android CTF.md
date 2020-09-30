# Android Challenges Hacker101

## H1 Thermostat(Easy, 2 flags)

Comenzando con el primer reto de Seguridad de aplicaciones mobiles en Android, se instala la aplicacion usando el **Android Debug Debugger(ADB)**

`adb install thermostat.apk`

Una vez instalado al abrir aparece la siguiente pantalla.

**Thermostat.apk**

![](/images/android/thermostat.png)

A continuacion, se decompila la aplicacion con **jadx** en su version de linea de comandos, hay que tener en consideracion que este metodo requiere de un directorio destino para almacenar el codigo fuente para este ejemplo sera ***thermostat***.

`jadx -d thermostat thermostat.apk `

Luego se abre este directorio con Android Studio, esto con la finalidad de poder manipular el codigo fuente, renombrar, modificar, entre otras opciones.

**Android Manifest**

![](/images/android/manifest.png)

En el manifiesto se revela que en cuanto permisos solo esta habilitado el permiso **INTERNET** que no representa mayor inconveniente, otras configuraciones que destacan son **android:usesCleartextTraffic** y **android:allowBackup** con valor **true** que significa que la aplicacion usa la comunicacion sin cifrar y ademas el usuario puede realizar el backup de la apk.

En cuanto a los componentes Thermostat.apk tiene una activity llamada *com.hacker101.level11.ThermosActivity* y posee un intent-filter. Un provider llamado *ProcessLifecycleOwnerInitializer* que esta exportado con valor **false**

**Payload Request**

![](/images/android/payloadrequest.png)

Se hace una llamada a la clase PayloadRequest , Volley facilita y permite el uso de redes en Android, permite transmitir datos a traves de la red por lo tanto para que este funcione debe estar habilitado el permiso **INTERNET**. Una solicitud sencilla se realiza usando el metodo **add()**, en la imagen se pasan mas parametros y se pasa la clase PayloadRequest junto a otras configuraciones. Examinando la clase PayloadRequest se encuentran las dos flags que necesita el reto.

**Flags_reto1**

![](/images/android/flags_reto1.png)

En esta clase se muestran las dos flags la primera se encripta con MD5 y luego se codifica con Base64 y se asigna al parametro X-MAC. La segunda flag se asigna a X-Flag y se guarda en texto plano.

Otro metodo para obtener la flag es usando BurpSuite, cuando se inicie la aplicacion esta herramienta capturara la peticion.

**Flags obtenidas en BurpSuite**

![](/images/android/burpsuitereto1.png)

## Intentional Exercise(Moderate, 1 Flag)

**Level13.apk**

De la misma manera que el reto anterior se decompila la aplicacion para proceder al analisis estatico.

![](/images/android/level13.png)

Se muestra un enlace que hace mencion a la flag, sin embargo al darle clic aparece que un mensaje diciendo que la solicitud no es valida.

![](/images/android/level132.png)

Como primer paso se mira en el manifiesto por informacion relevante que permita dar pautas sobre que pasos seguir en el analisis.

**Manifiesto de Android**

![](/images/android/manifest2.png)

En el manifiesto aparecen dos intent-filters **android:name:="android.intent.action.BROWSABLE"** esto corresponde a un Deep Link, un deep link es basicamente una URL que permite desplazarse por contenido especifico de la aplicacion.

Para hacer pruebas con los DeepLinks segun la documentacion de desarrollo de Android se puede usar el siguiente comando usando la herramienta **ADB**

`$ adb shell am start -W -a android.intent.action.VIEW -d <URI> <PACKAGE>`

Usando este comando se reemplazan URI y package por los respectivos valores en la aplicacion.

`adb shell am start -W -a "android.intent.action.VIEW" -d "http://level13.hacker101.com" com.hacker101.level13`

La activity MainActivity aparece en el manifiesto por lo que se considerara el punto de partida. Dentro de la clase MainActivity se muestra que la aplicacion crea un WebView y dos strings son declarados donde el mas representativo muestra una URL a la que apunta el WebView.

![](/images/android/level13mainactivity.png)

Si se ingresa esa URL en un navegador se obtiene el mismo resultado, la peticion no es valida.

![](/images/android/urlejecutada.png)

Examinando nuevamente el manifiesto data es la URL formada por el scheme y el host dando como resultado http://level13.hacker101.com , el metodo substring(28) ignora los primeros 28 caracteres y el resultado lo asigna a la variable **str2** para luego concatenarlo con **str**. En la otra condicion se chequea si el simbolo **?** existe en la cadena y de no ser asi lo agrega al final.

![](/images/android/mainactivity2.png)

En el ultimo bloque de codigo se crea un message digest usando el algoritmo de hash SHA-256, el hash se actualiza dos veces la primera con la key **s00p3rs3cr3tk3y**, y la segunda con el valor de str2, para finalmente construir la URL con str seguido de &hash=

Usando Burpsuite para analizar la peticion que pasa por la aplicacion se muestra lo siguiente.

![](/images/android/estructuraurl.png)

Dando a entender que la estructura de la URL seria `http://34.94.3.143/398abac4c8/appRoot?&hash="hash value"`, el valor de str2 es vacio seguido de **?**, luego &hash= y su valor.

Ingresando la peticion completa `http://34.94.3.143/35e3227bcf/appRoot?&hash=61f4518d844a9bd27bb971e55a23cd6cf3a9f5ef7f46285461cf6cf135918a1a` en el navegador produce el mismo resultado, inspeccionando el codigo fuente 

![](/images/android/codigofuente1.png)

En el codigo fuente se muestra que **/flagBearer** es parte de la URL, si se agrega esta parte la URL completa seria `http://34.94.3.143/35e3227bcf/appRoot/flagBearer?&hash=61f4518d844a9bd27bb971e55a23cd6cf3a9f5ef7f46285461cf6cf135918a1a`

![](/images/android/urlcorrecta.png)

A pesar que el valor de la URL es correcta se muestra el mensaje **Invalid hash**.

![](/images/android/urlincorrecta.png)

Para comprobar que efectivamente /flagBearer era el valor de str2 se reemplaza por cualquier cadena en este caso por **flag**

Recapitulando a pesar de tener la URL correcta el hash no es valido, esto se produce debido a que el metodo substring(28) elimina parte de la URL por lo que se pasa un valor vacio a str2 y al momento de realizar el hash SHA-256 este no encuentra nada en str2 produciendo un hash incorrecto.

`adb shell am start -W -a "android.intent.action.VIEW" -d "http://level13.hacker101.com/flagBearer" com.hacker101.level13`

Un metodo para poder bypassear esta validacion es a traves de Insufficient URL validation, cuando no existe una validacion correcta del origen de la URl se puede cargar una url arbitraria, mayor informacion en https://medium.com/bugbountywriteup/the-zaheck-of-android-deep-links-a5f57dc4ae4c. 


![](/images/android/flag_reto2.png)

Si se quiere generar el hash correcto se puede utilizar **Cyberchef** que es una herramienta que permite generar distintos de hash

![](/images/android/hashcorrecto.png)

Luego se agrega este nuevo hash a la URL dando como resultado la URL `http://34.94.3.143/35e3227bcf/appRoot/flagBearer?&hash=8743a18df6861ced0b7d472b34278dc29abba81b3fa4cf836013426d6256bd5e`

**NOTA**

Debido a que se expiro la sesion mientras se hacian las pruebas `http://34.94.3.143/35e3227bcf/` paso a ser `http://34.74.105.127/246febb4bb/`, sin embargo el resto de la URL permanece igual.

![](/images/android/flag_reto2_2.png)

Tambien se puede testear usando una prueba de concepto con un archivo .html y transferirlo a /sdcard y abrirlo directamente desde el dispositivo movil.

`adb push poc.html /sdcard/Download`

```html
<html>
	<a href="http://34.74.105.127/246febb4bb/appRoot/flagBearer?&hash=8743a18df6861ced0b7d472b34278dc29abba81b3fa4cf836013426d6256bd5e">test</a>
</html>
```

![](/images/android/poc1.png)












![](/images/android/bloque.png)






## Referencias

+ https://medium.com/bugbountywriteup/hacker101-ctf-android-challenge-writeups-f830a382c3ce



