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







