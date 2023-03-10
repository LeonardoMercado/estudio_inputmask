# Estudio de la Librería inputmask

Repositorio creado para el estudio y consulta de la [libreria inputmask](https://github.com/RobinHerbots/Inputmask#via-inputmask-class)

# Instalación:

> NOTA: Para este proyecto es necesario tener instalado [node.js](https://nodejs.org/es/) en su equipo local, se recomienda la version 14 o superior.

1. Clonar este repositorio en su equipo local:
    ~~~
    git clone https://github.com/LeonardoMercado/estudio_inputmask.git
    ~~~
2. Digirirse a la carpeta ```estudio_inputmask``` y descargar las dependencias necesarias para el proyecto con los comandos:
    ~~~ 
    cd estudio_inputmask
    npm install
    ~~~
3. Correr el proyecto con el comando: 
   ~~~
   npx http-server
   ~~~
4. Abrir en su explorador preferido la direccion: [localhost:8080](http://127.0.0.1:8080)

# Notas:


#### Definiciones de enmascaramiento predeterminadas:
Use algunas de estas definiciones para elaborar la mascara deseada:
- ```9``` : Para cualquier valor numerico.
- ```a``` : Para cualquier valor alfabético.
- ```*``` : Para cualquier valor alfanumerico.

Nota: Si necesita usar alguno de estos caracteres como valor estatico dentro de su mascara, escapelo con ```\\```.

#### Tipos de Enmascaramiento:
##### Mascaras Estaticas:
La máscara está definida y no cambiará durante la entrada. Por ejemplo:
~~~
$(document).ready(function(){
  $(selector).inputmask("aa-9999");
  $(selector).inputmask({mask: "aa-9999"});
});
~~~
La mascara ```aa-9999``` declara que solo admita dos valores alfabeticos, seguido por un guion y 4 numeros. Si declara la mascara de la siguiente manera: ```AA-9999```, el formato del valor alfabetico se pondra en mayusculas automaticamente. 

##### Mascaras Opcionales:
Es posible definir algunas partes en la máscara como opcionales. Esto se hace usando ```[ ]```.

~~~
$('#test').inputmask('(99) 9999[9]-9999');
~~~

Esta máscara permitirá entradas como (99) 12345-9999 o (99) 1234-9999. Note que el segundo conjunto de numeros pueden ser entre 4 y 5 numeros.

otros ejemplos:
- Input => 12123451234 con mascara => (12) 12345-1234 (completo)
- Input => 121234-1234 con mascara => (12) 1234-1234 (completo)
- Input => 1212341234 con mascara => (12) 12341-234_ (incompleto)

###### Mascaras Opcionales, opcion greedy:

Al definir una máscara opcional junto con la opción ```greedy: false```, la máscara de entrada mostrará primero la máscara más pequeña posible como entrada.

Por ejemplo, La mascara definica de la siguiente manera:
~~~
Inputmask({
    mask: "9[-9999]", 
    greedy: false,
  }).mask("#basicoPruebas");
~~~

mostrará:
![input sin greedy](https://i.imgur.com/OZ6d9Ra.png)

Pero al definir la opcion de ```greedy: true``` 
~~~
Inputmask({
    mask: "9[-9999]", 
    greedy: true,
  }).mask("#basicoPruebas");
~~~

mostrará:
![input sin greedy](https://i.imgur.com/Od86niN.png)

Por defecto la opcion de ```greedy``` esta en false.

#### Máscaras dinámicas:

Las máscaras dinámicas pueden cambiar durante la entrada. Para definir una parte dinámica use ```{ }```.

{n} => n repeticiones {n|j} => n repeticiones, con j [jitmasking](#jitMasking) {n,m} => de n a m repeticiones {n,m|j} => de n a m repeticiones, con j [jitmasking](#jitMasking)

También se permiten {+} y {*}. + empezar desde 1 y * empezar desde 0.

Por ejemplo, la mascara definida como:
~~~
Inputmask({
    mask: "aa-9{4}",
  }).mask("#basicoPruebas");
~~~

Se debera ingresar dos caracteres alfabeticos seguidos de un guion y 4 numeros.

- Input => ab3232 con mascara => ab-3232 (completo)
- Input => cd-1234 con mascara => cd-1234 (completo)
- Input => ef234 con mascara => ef-234_ (incompleto)

Otra forma de ver esta mascara es: ```aa-9999```, este es el equivalente de esta mascara dinamica en mascara estatica.

Otro ejemplo:
~~~
Inputmask({
    mask: "aa-9{1,4}",
  }).mask("#basicoPruebas");
~~~

Se debera ingresar dos caracteres alfabeticos seguidos de un guion y entre 1 hasta 4 numeros.

- Input => ab3232 con mascara => ab-3232 (completo)
- Input => cd-12 con mascara => cd-12 (completo)
- Input => ef con mascara => ef-_ (incompleto)

Otra forma de ver esta mascara es: ```aa-9[9][9][9]```, este es el equivalente de esta mascara dinamica en mascara opcional.

#### Máscaras de alternador:

La sintaxis del alternador es como una declaración OR. La máscara puede ser una de las opciones especificadas en el alternador.

Para definir un alternador use el ```|```. Por ejemplo: 
- ```"a|9"``` => a o 9 
- ```"(aaa)|(999)"``` => aaa o 999 
- ```"(aaa|999|9AA)"``` => aaa o 999 o 9AA
- ```"aaaa|9999"``` => aaa a o 9 999

Ejemplo:
~~~
Inputmask({
    mask:"(99.9)|(X)",
    definitions: {
      "X": {
        validator: "[xX]",
        casing: "upper"
      }
    }
  }).mask("#alternadorEjemplo");
~~~

Se debe ingresar dos numeros seguidos de un punto y otro numero, **O** ingresar una X. La definicion dentro de la declaracion, valida que si se ingresa una x o X se tome como entrada y se convierte a MAYUSCULAS.

Otra forma de declara esta mascara es:
~~~
Inputmask({
    mask: ["99.9", "X"],
    definitions: {
      "X": {
        validator: "[xX]",
        casing: "upper"
      }
    }
  }).mask("#basicoPruebas");
~~~

#### Máscaras de Preprocesamiento:

Puede definir la máscara como una función que le permite preprocesar la máscara resultante. Ejemplo de clasificación de varias máscaras o recuperación de definiciones de máscaras dinámicamente a través de ajax. El preprocesamiento de la función debe devolver una definición de máscara válida. Es decir, podemos ejecutar una funcion dentro de la definicion del objeto tipo mascara el cual debe devolver una mascara valida para el input.

Ejemplo:
~~~
const MAX_VALUE_CO = 3;
const MAX_VALUE_OTHERS = 6;
const PAIS = 'CO';
Inputmask({
  mask: function () { 
    let mascara = '';
    let paisActual = $('#mascaraFuncion').attr('data');
    if(PAIS === paisActual){
      mascara = `AAA-9{${MAX_VALUE_CO}}`;
    } else {
      mascara = `AAA-9{${MAX_VALUE_OTHERS}}`;
    }
    return [mascara]; 
  }
}).mask("#mascaraFuncion");
~~~

Si la data del input ```mascaraFuncion``` es CO se mostrará:
![mascaraCO](https://i.imgur.com/0tdg3jP.png)

Caso contrario mostrará:
![mascaraCO](https://i.imgur.com/K692HTb.png)

<a name="jitMasking"></a>
#### Enmascaramiento JIT:

Enmascaramiento justo a tiempo (JIT => Just In Time). Con la opción ```jitMasking```, puede habilitar el enmascaramiento jit. La máscara solo será visible para los caracteres ingresados por el usuario. de manera predeterminada ```jitMasking``` es falso. El valor puede ser ```true```, ```false```, o un número de umbral, por ejemplo ```5```.

Es decir, el atributo ```jitMasking``` controla si la mascara se muestra por defecto o no, tambien se puede mostrar hasta una cantidad de caracteres definidos por el umbral pasado como parámetro.

Por ejemplo:
~~~
Inputmask({
  mask: "999-999-9999",
  jitMasking: false,
}).mask("#mascaraJIT");
~~~

mostrará:
![jitMaskingFalse](https://i.imgur.com/6VYbsv3.png)

Pero si activamos el actributo jitMasking:
~~~
Inputmask({
  mask: "999-999-9999",
  jitMasking: true,
}).mask("#mascaraJIT");
~~~

mostrará:
![jitMaskingTrue](https://i.imgur.com/uAaJa4z.png)

y al ingresar valores en el input se aplicara la mascara al momento de ingresar cada valor, de forma que se verá así:
![jitMaskingFull](https://i.imgur.com/MoWFNLy.png)


Tambien podemos definir un umbral para que la mascara se renderize, y despues de dicho umbras sera just in time:

Por ejemplo:

~~~
Inputmask({
  mask: "999-999-9999",
  jitMasking: 5,
}).mask("#mascaraJITUmbral");
~~~

mostrará:
![jitmaskingUmbral](https://i.imgur.com/3ZpEecX.png)

#### Opciones:

##### PlaceHolder
Cambie el marcador de posición de la máscara. Por defecto: "_"

En lugar de "_", puede cambiar la máscara de caracteres vacíos como desee, simplemente agregando la opción de marcador de posición.
Por ejemplo, el marcador de posición: " " cambiará el autorrelleno predeterminado con valores vacíos

Ejemplo:
~~~
Inputmask({
  mask: '99/99/9999',
  placeholder: "dd/mm/yyyy"
}).mask("#opcionesPlaceHolder");
~~~

Mostrará:
![mascaraPlaceHolder](https://i.imgur.com/qH9uxDa.png)

##### oncomplite y onincomplete:
La opción **oncomplite** se ejecuta cuando la mascara esta completa, y **onincomplete** cuando la mascara no esta completa.

Ejemplo:
~~~
Inputmask({
  mask: 'a{3,9}',
  oncomplete: function(){ 
    mostradorError($('#errorOnComplete'),true); 
  },
  onincomplete: function(){
    mostradorError($('#errorOnComplete'),false); 
  }
}).mask("#opcionesOnComplite");

function mostradorError(select,esValido){
  select.removeClass('d-none');
  select.text(`El campo ${(esValido)?'':'NO'} es valido`);
  if(esValido){
    select.addClass('text-success');    
    select.removeClass('text-danger');    
  } else {
    select.removeClass('text-success');    
    select.addClass('text-danger');  
  }
}
~~~

Cuando la mascara este COMPLETA mostrará:
![mascaraCompleta](https://i.imgur.com/yAf7mGY.png)
Cuando la mascara este INCOMPLETA mostrará:
![mascaraCompleta](https://i.imgur.com/XWt5ixK.png)

Estos dos metodos son necesarios para realizar la validacion de los inputs antes de hacer el submit del formulario.

##### oncleared:
La opcion **oncleared** se ejecutará cuando el usuario borre el contenido del input.

Ejemplo:
~~~
Inputmask({
    mask: 'a{3}-9{3}',
    oncleared: function(){ alert('Debe ingresar un valor para este input'); }
  }).mask("#opcionesCleared");
~~~

Nota: La funcion definida en la opción se ejecuta antes de borrar el último carácter del input.
![mascaraCleared](https://i.imgur.com/iRzn9hP.png)

Esta opcion puede ser util para:
1. Mostrar un mensaje de error: Puedes usar "oncleared" para mostrar un mensaje de error cuando un usuario borra el contenido de un campo de entrada que requiere un valor.
2. Cambiar el estado de un elemento: Si deseas cambiar el estado de un elemento cuando se borra el contenido de un campo de entrada.

Recuerde que también puede usar la opción **onincomplete** para generar el comportamiento deseado, tanto si se borra todo el contenido como si no se completa la restricción del input.

##### clearIncomplete:
Esta opción borra todo el contenido del input si no se completa la mascara.

Ejemplo:
~~~
Inputmask({
  mask: '+(57) 9{3} 9{3} 9{4}',
  "clearIncomplete": true
}).mask("#opcionesClearIncomplete");
~~~

Nota: Esta opción debe evitarse ya que puede ser molesto para el usuario escribir todo el dato de nuevo cada vez que no se complete el input. Es mejor avisarle y permitir modificaciones.

#### Alias:
Con un alias, puede definir una definición de máscara compleja y llamarla utilizando un nombre de alias. Esto es principalmente para simplificar el uso de sus máscaras. Algunos alias que se encuentran en las extensiones son correo electrónico, moneda, decimal, entero, fecha, fecha y hora, dd/mm/yyyy, etc.

Primero, debe crear una definición de alias. La definición de alias puede contener opciones para la máscara, definiciones personalizadas, la máscara a usar, etc.

Cuando pasa un alias, primero se resuelve el alias y luego se aplican las otras opciones. Entonces puede llamar a un alias y pasar otra máscara para que se aplique sobre el alias. Esto también significa que puede escribir alias que "hereden" de otro alias.

##### datetime:
Permite generar una mascara tipo fecha y hora muy facilmente, con esta máscara, se puede garantizar que los datos de fecha y hora se ingresen en el formato correcto, lo que facilita su posterior procesamiento y análisis.

Ejemplo:
~~~
Inputmask({
  alias: "datetime",
}).mask("#aliasDateTime");
~~~

Mostrará:
![datetime1](https://i.imgur.com/7zUWOvH.png)
![datetime2](https://i.imgur.com/BSttmu0.png)

##### decimal:
Con esta máscara, se puede garantizar que los números decimales se ingresen en el formato correcto, lo que facilita su posterior procesamiento y análisis. Además, también se pueden establecer opciones como la cantidad de dígitos decimales permitidos y el separador de decimales.

Ejemplo:
~~~
Inputmask({
  alias: "decimal",
  prefix: '$ ',
  radixPoint: ",",
  autoGroup: true,
  groupSeparator: "\.",
  digits: 2,
  digitsOptional: false,
  placeholder: "0",
  oncomplete: function(){
    console.log("La máscara se ha completado");
  },
  onincomplete: function(){
    console.log("La máscara no se ha completado");
  }
}).mask("#aliasDecimal");
~~~

Mostrará:
![decimal1](https://i.imgur.com/u74GtOX.png)
![decimal2](https://i.imgur.com/iqGOzu9.png)
![decimal3](https://i.imgur.com/jiTYkH7.png)
![decimal4](https://i.imgur.com/R57pWbe.png)
![decimal4](https://i.imgur.com/zvUKYw1.png)

##### integer:

La máscara se aplica a un elemento de entrada y limita la entrada del usuario a solo dígitos numéricos. Es útil para asegurarse de que los datos de entrada sean números enteros válidos. Por ejemplo, se puede utilizar para aplicar una máscara de entrada a un elemento de formulario que requiera una edad, una identificación o un número de teléfono.

Ejemplo:
~~~
Inputmask({
  alias: "integer",
}).mask("#aliasInteger");
~~~

![integer1](https://i.imgur.com/9w5sPxq.png)
![integer2](https://i.imgur.com/lCj5IJ5.png)


##### currency:

La máscara se aplica a un elemento de entrada y limita la entrada del usuario a un formato de moneda específico, que se puede personalizar mediante opciones adicionales. Es útil para asegurarse de que los datos de entrada sean valores de moneda válidos.

Ejemplo moneda Colombiana:
~~~
Inputmask({
  alias: "currency",
  radixPoint: ",",
  prefix: "COP$ ",
  autoGroup: true,
  groupSeparator: "\.",
  digits: 2,
  digitsOptional: false,
  placeholder: "0"
}).mask("#aliasCurrencyCO");
~~~

Mostrará:
![currencyCO1](https://i.imgur.com/UW4GDUX.png)
![currencyCO2](https://i.imgur.com/A4nTnii.png)

Ejemplo moneda Mexicana:
~~~
Inputmask({
  alias: "currency",
  radixPoint: ",",
  prefix: "MXN$ ",
  autoGroup: true,
  groupSeparator: "\.",
  digits: 2,
  digitsOptional: false,
  placeholder: "0"
}).mask("#aliasCurrencyMX");
~~~

Mostrará:
![currencyMX1](https://i.imgur.com/LpiN1D2.png)
![currencyMX2](https://i.imgur.com/o0wp1Y6.png)

##### email:
Se utiliza para aplicar una máscara de entrada para validar los valores de correo electrónico. La máscara se aplica a un elemento de entrada y limita la entrada del usuario a un formato de correo electrónico válido, que es [nombre de usuario] @ [servidor de correo].

Ejemplo:
~~~
Inputmask({
  alias: "email",
}).mask("#aliasEmail");
~~~

Mostrará
![email1](https://i.imgur.com/Xvjbh9L.png)
![email2](https://i.imgur.com/Ppjf2T7.png)

##### ip:
Se utiliza para aplicar una máscara de entrada para validar los valores de direcciones IP. La máscara se aplica a un elemento de entrada y limita la entrada del usuario a un formato de dirección IP válido, que consiste en cuatro números de 8 bits separados por puntos.

Esta máscara se asegura de que el usuario solo pueda ingresar una dirección IP válida en el formato [número]. [número]. [número]. [número].


Ejemplo:
~~~
Inputmask({
  alias: "ip",
}).mask("#aliasIp");
~~~

![ip1](https://i.imgur.com/UFYIvd2.png)
![ip2](https://i.imgur.com/WhPXTKR.png)

##### mac:
Se utiliza para aplicar una máscara de entrada para validar los valores de direcciones MAC. La máscara se aplica a un elemento de entrada y limita la entrada del usuario a un formato de dirección MAC válido, que consiste en seis números o letras pares separados por dos puntos.

Esta máscara se asegura de que el usuario solo pueda ingresar una dirección MAC válida en el formato [número o letra][número o letra]:[número o letra][número o letra]:[número o letra][número o letra]:[número o letra][número o letra]:[número o letra][número o letra]:[número o letra][número o letra].

Ejemplo:
~~~
Inputmask({
  alias: "mac",
}).mask("#aliasMac");
~~~

![mac1](https://i.imgur.com/IPlkJB9.png)
![mac2](https://i.imgur.com/gC1MY6F.png)

##### url:
Se utiliza para aplicar una máscara de entrada para validar las direcciones URL. La máscara se aplica a un elemento de entrada y limita la entrada del usuario a un formato de dirección URL válido, que incluye un esquema (por ejemplo, "http://" o "https://") seguido de un nombre de dominio o dirección IP y una ruta opcional.

Esta máscara se asegura de que el usuario solo pueda ingresar una dirección URL válida en el formato ```[esquema]://[nombre de dominio o dirección IP]/[ruta opcional]```.

Ejemplo:
~~~
Inputmask({
  alias: "url",
}).mask("#aliasUrl");
~~~

![url1](https://i.imgur.com/O1JxuTi.png)
![url2](https://i.imgur.com/6tutWRz.png)
![url3](https://i.imgur.com/pnUXmgV.png)

Resumen alias de la libreria:
| Alias        | Descripción                                                                                                                                                                                                                                                                                            |
|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| decimal      | Permite ingresar números decimales con un punto decimal.                                                                                                                                                                                                                                            |
| integer      | Permite ingresar solo números enteros.                                                                                                                                                                                                                                                              |
| alphanumeric | Permite ingresar caracteres alfanuméricos (letras y números).                                                                                                                                                                                                                                   |
| currency     | Permite ingresar números con formato moneda, con un símbolo de moneda opcional y separador de miles opcional.                                                                                                                                                                                  |
| email        | Valida que el input sea un email válido                                                                                                                                                                                                                                                         |
| ip           | Permite ingresar una dirección IP v4.                                                                                                                                                                                                                                                             |
| mac          | Permite ingresar una dirección MAC.                                                                                                                                                                                                                                                              |
| url          | Permite ingresar una url válida.                                                                                                                                                                                                                                                                 |
| datetime         | Permite ingresar una fecha con un formato específico.                                                                                                                                        



# Conclusiones:

Existen tres caracteres especiales para definir las mascaras, los cuales son:
- ```9```:Permite ingresar cualquier valor numerico.
- ```a``` | ```A```:Permite ingresar cualquier valor alfabetico. | permite ingresar cualquier valor alfabetico y lo convierte a MAYUSCULAs.
- ```*```:Permite ingresar cualquier valor alfanumerico.

Si se necesitan estos caracteres para una mascara en particular, se deben escapar con ```\\```

# Autor
[**Ing. Leonardo Fabio Mercado Benítez**](https://www.linkedin.com/in/leonardofabiomercadobenitez/)
