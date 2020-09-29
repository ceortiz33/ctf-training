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










## Referencias

+ https://medium.com/bugbountywriteup/hacker101-ctf-android-challenge-writeups-f830a382c3ce



