# Injured Android CTF

![](/images/injuredandroid/img1.png)

InjuredAndroid es una aplicacion que sirve como entrenamiento para el analisis de vulnerabilidades en Android y Bug Bounty de dispositivos mobiles, algunos de los retos que presenta son XSS, ataques mediante Deeplinks, API credentials, entre otros.

## XSS Test

Como primer reto esta **XSS Test** dentro de ella se muestra un campo para ingresar datos.

![](/images/injuredandroid/img2.png)

Si usamos uno de las cadenas mas comunes para probar XSS como `<script>alert('XSS')</script>` muestra el siguiente resultado.

![](/images/injuredandroid/img3.png)

El reto consiste en mostrar que es vulnerable a XSS, ademas motiva al usuario a mirar dentro del codigo fuente de la activity "DisplayPostXSS". El codigo fuente se muestra en la siguiente imagen.

![](/images/injuredandroid/img4.png)

Examinando el codigo encontramos que se declara una webview pero ademas de las configuraciones como que se ejecute el navegador de Chrome, se utiliza `setJavaScriptEnabled(true)` que es una opcion vulnerable porque habilita la ejecucion de codigo Javascript.

## Flag One Login

![](/images/injuredandroid/img5.png)

Para superar este reto debemos ingresar la flag correspondiente y enviarla para ganar acceso, en el triangulo rojo nos muestra pistas como 'La flag esta en tu nariz' y 'La Flag tambien esta en la GUI', en el codigo fuente de la clase FlagOneActivity se muestra una string en texto plano  que presumiblemente seria la flag.

![](/images/injuredandroid/img6.png)

Al ingresar esa cadena superamos el reto FlagOneLogin

![](/images/injuredandroid/img7.png)

## Flag Two: Exported Activity

Cuando una activity esta exportada como **true** esto implica que dicha activity puede ser accedida por otras aplicaciones, esto no necesariamente implica un riesgo de seguridad si ese es el proposito o si se aplican ciertos permisos para que solo tengan acceso ciertas aplicaciones. En este reto nos sugiere que hallemos la manera de pasar a traves de la aplicacion e invocar otras activities, para esto el mismo reto sugiere que se enfoque en los terminos **exported** y **activities puede ser ingresados mediante adb o drozer**.

En el manifiesto de Android se puede encontrar informacion util que sirve como guia para posibles ataques, como resultado de la busqueda se obtuvo la activity b25lActivity, TestBroadcastReceiver y MainActivity que tiene un intent-filter que por defecto se exporta como true.

![](/images/injuredandroid/img8.png)

Usando el comando con adb se puede saltar acceder al contenido de la flag.

`adb shell am start -n b3nac.injuredandroid/b3nac.injuredandroid.b25lActivity`

![](/images/injuredandroid/img9.png)

## Flag Three: Resources

En este reto se solicita una flag para superarlo. Entre las pistas que nos proporcionan para el reto esta que se buscan dentro de los archivos .xml

![](/images/injuredandroid/img10.png)

Siguiendo la recomendacion buscamos el archivo strings.xml localizado en resources.arsc/res/values/strings.xml y encontrando la flag en uno de los strings declarados para la aplicacion.

![](/images/injuredandroid/img11.png)

Ingresamos esta flag y aparece una ventana mencionando que hemos logrado superar el reto.

![](/images/injuredandroid/img12.png)

## Flag Four - Login 2

Al igual que el primer reto el objetivo es conseguir la flag y de esta manera superarlo. 

![](/images/injuredandroid/img13.png)

Examinamos el codigo fuente de la clase FlagFourActivity y en la funcion submitFlag nos muestra como declara un arreglo de bytes usando la funcion g().a()

![](/images/injuredandroid/img14.png)

En este metodo se utiliza Base64decode a una cadena de caracteres.

![](/images/injuredandroid/img15.png)

Podemos usar Cyberchef para que decodifique el contenido de la cadena de caracteres dando como resultado la flag del reto 4.

![](/images/injuredandroid/img16.png)

## Flag Five: Exported Broadcast Receiver

A diferencia de los demas retos, aqui no se muestra ningun texto ni campo que llenar, solo aparece un mensaje cada vez que entramos.

![](imagesinjuredandroid/img17.png)

La Flag aparece luego de varios intentos de ingresar. Sin embargo, para buscar una explicacion a este comportamiento revisamos el codigo fuente en busca de mayor informacion

La clase FlagFiveActivity hace una llamada a FlagFiveReceiver()

![](/images/injuredandroid/img18.png)

FlagFiveReceiver extiende BroadcastReceiver, ademas comprobamos que el mensaje mostrado depende del numero de intentos realizados para esto usamos el contador i que al incrementar su valor va mostrando nuevos mensajes, cuando el contador llega a 2 se muestra la flag

![](/images/injuredandroid/img19.png)

El mensaje final se produce concatenando "You are a winner" con el resultado de la funcion k.a(Zkdlt0WwtLQ=), investigando mas a profundidad la funcion k.a realiza un proceso de conversion, a() recibe como parametro una cadena de texto y posteriormente utiliza el algoritmo DES para encriptar.

![](/images/injuredandroid/img20.png)

La imagen anterior muestra dos arrays de bytes, si exploramos el contenido de esos metodos se muestra dos keys utilizadas para la encriptacion DES, particularmente se utiliza solo el primero ya que la funcion **h.b()** retorna f1911a que es el array utilizado en **k.a()**

![](/images/injuredandroid/img21.png)

Con la ayuda de Cyberchef podemos averiguar cual era la key que se utilizaba para la encriptacion DES.

![](/images/injuredandroid/img22.png)

## Flag Six: Login 3

En este reto tambien se utiliza el metodo k.a() que realiza la conversion de un string base64 a encriptacion DES, para superar este reto debemos ingresar esta flag en el campo correspondiente y lograr que aparezca la activity de reto superado. Frida es una herramienta que nos permite realizar llamadas a funciones durante el analisis dinamico, una de sus funciones mas comunes es realizar hooking de metodos y funciones para modificar componentes durante la ejecucion de la aplicacion.

Creamos un script Javascript que permita capturar la funcion k.a() y modificar el parametro string que recibe cuando es ejecutada.

```javascript
//flag6.js

console.log("Script loaded succesfully");

Java.perform(function x() {
	console.log("Inside java perform function");
	//Class the function belongs to.
	var my_class = Java.use("b3nac.injuredandroid.k");
	// Hook the function with parameter string
	var string_class = Java.use("java.lang.String");
	my_class.a.overload("java.lang.String").implementation = function(x){
		console.log("**************************************");
		//Create a new String and call the function with the new input
		var my_string = string_class.$new("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==");
		console.log("Original arg: " + x);
		var ret = this.a(my_string)
		console.log("Return value: " + ret);
		console.log("***************************************")
		return ret;
	};

//Find an instance of the class and call "decrypt"
	Java.choose("b3nac.injuredandroid", {
		onMatch: function(instance) {
			console.log("Found instance: " + instance);
			console.log("Result of decrypt func: " + instance.a());
		},
		onComplete: function () { }	
	});
});
```

Luego tenemos que llamar este archivo javascript desde un script en python teniendo el **frida-server** habilitado.

```python
#flag6.py

import frida
import time

device = frida.get_usb_device() # get android device
pid = device.spawn(["b3nac.injuredandroid"])
device.resume(pid)
time.sleep(1) # without it Java perform fails
session = device.attach(pid)
script = session.create_script(open("flag6.js").read())
script.load()

# Prevent the python script from terminating

input()

```

Ejecutamos el script usando `python flag6.py` e ingresamos cualquier texto en el campo de flagSixActivity, este se reemplazara con el string que agregamos en el script dando como resultado la flag del reto.

![](/images/injuredandroid/img23.png)

## Flag Seven: SQLite

En este reto nos piden que ingresemos tanto la flag como una password, como consejos nos dicen que ejecutemos adb como root y que nos quedemos en esta activity

![](/images/injuredandroid/img24.png)

Inspeccionando el codigo de la activty FlagSevenActivity encontramos lo siguiente, se generan dos campos para la base de datos **title** y **subtitle**, ademas se usa **Base64.decode** para descifrar el contenido de strings. Recordando uno de las pistas que se mencionaban en el reto "stay on this activity", hace referencia al metodo OnDestroy() ya que si salimos de la aplicacion se destruye la base de datos **Thisisatest.db**.

![](/images/injuredandroid/img25.png)

![](/images/injuredandroid/img26.png)

Las bases de datos normalmente se almacenan dentro del directorio `/data/data/[nombre de la aplicacion]/databases`, aqui se encuentra la base de datos **Thisisatest.db**

![](/images/injuredandroid/img27.png)

Usando adb pull podemos extraer la base de datos desde el dispositivo hasta nuestra PC, luego se puede ver el contenido usando **DB Browser for SQLite**

![](/images/injuredandroid/img28.png)

![](/images/injuredandroid/img32.png)

![](/images/injuredandroid/img29.png)

![](/images/injuredandroid/img30.png)

![](/images/injuredandroid/img31.png)






