# Micro-Biblioteca: Ejemplo Práctico de Microservicios

Este repositorio contiene un ejemplo simple de una  biblioteca virtual construida usando una **arquitectura de microservicios**.

El ejemplo fue diseñado para ser usado en una  **aula práctica sobre microservicios**, que puede, por ejemplo, ser realizada después de conocer conceptos básicos de la materia.

El objetivo de esta clase es permitir que el alumno tenga un primer contacto con microservicios y con tecnologías normalmente usadas en este tipo de arquitecturas, tales como **Node.js**, **REST**, **gRPC** y **Docker**.

Como nuestro objetivo es didáctico, en la biblioteca virtual se encuentra a la venta solo tres libros, como se ve en la próxima figura, que muestra la interfaz Web del sistema. Además, la operación de compora solo simula la acción del usuario, no efectuando cambios en el stock. Así, los clientes de la biblioteca pueden realizar solo dos operaciones: (1) listar los productos en venta; (2) calcular el flete de envio.

<p align="center">
    <img width="70%" src="https://github.com/dsanmartins/micro-biblioteca/blob/main/resources/sitioweb.png" />
</p>

En el documento restante vamos:

-   Describir el sistema, con foco en la arquitectura.
-   Presentar instrucciones para la ejecución local, usando el código disponible en el repositorio.
-   Describir dos tareas prácticas para que sean realizadas por los alumnos, las cuales involucran:
    - Tarea Práctica #1: Implementación de una nueva operación en uno de los microservicios
    - Tarea Práctica #2: Creación de contenedores Docker para facilitar la ejecución de los microservicios.

## Arquitectura

La micro-biblioteca posee cuatro microservicios:

-   Front-end: microservicio responsable por la interfaz con el usuario, como fue mostrado en la figura anterior.
-   Controller: microservicio responsable por intermediar la comunicación entre el front-end y el backend del sistema.
-   Shipping: microservicio para cálculo de flete.
-   Inventory: microservicio para control del stock de la biblioteca.

Los cuatro microservicios están implementados en **JavaScript**, usando  Node.js para para la ejecución de los servicios back-end.
Sin embargo, **usted conseguirá completar las tareas prácticas aun si nunca programó en JavaScript**. El motivo es que nuestras instrucciones ya incluyen las partes de código que deben ser copiados en el sistema.

Para facilitar la ejecución y entendimiento del sistema, también no usamos base de datos o servicios externos.

## Protocolos de Comunicación

Como se ilustra en el siguiente diagrama, la comunicación entre el front-end y el backend usa una **API REST**, como es común en el caso de un sistema Web.

La comunicación entre el Controller y los microservicios del back-end está basada en [gRPC](https://grpc.io/).

<p align="center">
    <img width="70%" src="https://github.com/dsanmartins/micro-biblioteca/blob/main/resources/arquitectura.png" />
</p>

Optamos por usar gRPC en el back-end porque este posee un desempeño mejor que el de REST. Especificamente, gRPC está basado en el concepto de **Llamada Remota de Procedimientos (RPC)**. La idea es simple: en aplicaciones distribuídas que usan gRPC, un cliente puede llamar funciones implementadas en otros procesos de forma transparente, esto es, como si tales funciones fuesen locales. En otras palabras, llamadas gRPC tienen la misma sintaxis de llamas normales de funciones.

Para viabilizar esta transparencia, gRPC usa dos conceptos centrales:

-   un lenguaje para la definición de interfaces
-   un protocolo para el intercambio de mensajes entre aplicaciones clientes y servidores.


Especificamente, en el caso de gRPC, la implementación de estos dos conceptos ganó el nombre de **Protocol Buffer**. Osea, podemos decir que:

> Protocol Buffer = lenguaje para definición de interfaces + protocolo para definición de los mensajes intercambiados entre aplicaciones clientes y servidores

### Ejemplo de Archivo .proto

Cuando trabajamos con gRPC, cada microservicio posee un archivo `.proto` que define la firma de las operaciones que este pone a disposición para los otros microservicios.
En este archivo, declaramos también los tipos de los parámetros de entradas y salida de esas operaciones.

El ejemplo a continuación muestra el archivo [.proto](https://github.com/dsanmartins/micro-biblioteca/blob/main/proto/shipping.proto) de nuestro microservicio flete. En este, definimos que este microservicio disponibilize una función `GetShippingRate`. Para llamar esa función debemos pasar como parámetro de entrada un objeto conteniendo el código postal (`ShippingPayLoad`). Después de su ejecución, la función retorna como resultado otro objeto (`ShippingResponse`) con el valor del flete.

<p align="center">
    <img width="70%" src="https://github.com/dsanmartins/micro-biblioteca/blob/main/resources/descripcion.png" />
</p>

En gRPC, los mensajes (ejemplo: `Shippingload`) son formados por un conjunto de campos, tal como en un `struct` del lenguaje C. Todo campo posee un nombre (ejemplo: `cep`) y un tipo (ejemplo: `string`). Además,  todo campo tiene un número entero que funciona como un identificador único para el mismo, en el mensaje (ejemplo: ` = 1`). Este número es usado por la implementación de gRPC para identificar el campo en el formato binário de datos usado por gRPC para comunicación distribuída.

Archivos .proto son usados para generar **stubs**, que mas que nada son proxies que encapsulan los detalles de comunicación en red, incluyenndo intercambio de mensajes, protocolos, etc. Mas detalles sobre el patrón de diseño Proxy puede ser obtenido en libros de ingeniría de software.

En lenguajes estáticos, normalmente, se necesita llamar un compilador para generar el código de tales stubs. En el caso de JavaScript, este paso no es necesario porque los stubs son generados de forma transparente, en tiempo de ejecución.

## Ejecutando el Sistema

A seguir vamos a describir la secuencia de pasos para ejecutar el sistema localmente en su computador. Osea, todos los microservicios estarán corriendo en su máquina.

**IMPORTANTE:** Usted debe seguir estos pasos antes de implementar las tareas prácticas descritas en las próximas secciones.

1. Haga un fork del repositório. Para esto, basta hacer click en botón **Fork** en la esquna superior derecha de esta página.

2. Dirigirse al terminal de su sistema operativo y haga un clon del proyecto (acuérdese de incluir su usuario GitHub en la URL antes de ejecutar)

```
git clone https://github.com/<SU USUÁRIO>/micro-biblioteca.git
```

3. También es necesario tener Node.js instalado en su máquina. Se usted no lo tiene, siga las instrucciones para la instalación en esta página [página](https://nodejs.org/en/download/).

4. En un terminal, vaya al directorio en el cual su proyecto fue clonado e instale las dependencias necesarias para la ejecución de los microservicios:

```
cd micro-biblioteca
npm install
```

5. Inicie los microservicios a través del comando:

```
npm run start
```

6. Para fines de testing, efectue una petición al microservicio responsable por la  API del back-end.

-   Si tiene el programa `curl` instalado en su máquina, basta usar:

```
curl -i -X GET http://localhost:3000/products
```

-   En caso contrario, usted puede hacer una petición accesando, en su navegador, a la seguiente URL: `http://localhost:3000/products`.

7. Pruebe ahora el sistema entero, abriendo el front-end en un navegador: http://localhost:5000. Haga entonces un testing de las principales funcionalidades de la biblioteca.

## Tarea Práctica #1: Implementando una Nueva Operación

En esta primeira tarea, usted irá a implementar una nueva operación en el servicio `Inventory`. Esta operación, llamada `SearchProductByID` va a buscar un produto, dado su ID.

Como fué descrito anteriormente, las firmas de las  operaciones de cada microservicio son definidas en un archivo `.proto`, en el caso [proto/inventory.proto](https://github.com/dsanmartins/micro-biblioteca/blob/main/proto/inventory.proto).

#### Paso 1

Primero, usted debe declarar la firma de la nueva operación. Para esto, incluya la definición de esta firma en el archivo `.proto` (en la línea siguiente a la firma de la función `SearchAllProducts`):

```proto
service InventoryService {
    rpc SearchAllProducts(Empty) returns (ProductsResponse) {}
    rpc SearchProductByID(Payload) returns (ProductResponse) {}
}
```

En otras palabras, usted está definiendo que el microservicio `Inventory` va a responder a una nueva petición, llamada `SearchProductByID`, que tiene como parámetro de entrada un objeto del tipo `Payload` y como parámetro de salida un objeto del tipo `ProductResponse`.

#### Passo 2

Incluya también en el mismo archivo la declaración del tipo del objeto `Payload`, el qual solo contiene el ID del producto a ser buscado.

```proto
message Payload {
    int32 id = 1;
}
```

Vea que `ProductResponse` -- esto es, el tipo de retorno de la  operación -- ya está declarado mas abajo en el archivo `proto`:

```proto
message ProductsResponse {
    repeated ProductResponse products = 1;
}
```

Osea, la respuesta de nuestra petición contendrá un único campo, del tipo `ProductResponse`, que también ya está implementado en el mismo archivo:

```proto
message ProductResponse {
    int32 id = 1;
    string name = 2;
    int32 quantity = 3;
    float price = 4;
    string photo = 5;
    string author = 6;
}
```

#### Paso 3

Ahora usted debe implementar una función `SearchProductByID` en el archivo  [services/inventory/index.js](https://github.com/dsanmartins/micro-biblioteca/blob/main/services/inventory/index.js).

Reforzando, en el paso anterior, solo declaramos la firma de esa función. Entonces, ahora, vamos a proporcionar una implementación para ella.

Para esto, usted necesitará implementar la función requerida por el segundo parámetro de la función `server.addService`, localizada en la linea 17 del archivo [services/inventory/index.js](https://github.com/dsanmartins/micro-biblioteca/blob/main/services/inventory/index.js).


De forma semejante la  función `SearchAllProducts`, que ya está implementada, usted debe adicionar el cuerpo de la  función `SearchProductByID` con la lógica de búsqueda de produtos por ID. Este código debe ser adicionado luego después de `SearchAllProducts` en la linea 23.

```js
    SearchProductByID: (payload, callback) => {
        callback(
            null,
            products.find((product) => product.id == payload.request.id)
        );
    },
```

La función anterior usa el método `find` para buscar en `products`  por el ID de producto proporcionado. Ver que:

-   `payload` es el parámetro de entrada de nuestro servicio, conforme fue definido antes en el archivo .proto (paso 2). El almancena el ID del producto que queremos buscar. Para accesar este ID basta escribir `payload.request.id`.

-   `product` es una unidad de producto a ser buscado por la función `find` (nativa de JavaScript). Esta búsqueda es realizada en todos los items de la lista de produtos hasta que un primer `product` atienda la condición de búsqueda, esto es `product.id == payload.request.id`.

-   [products](https://github.com/dsanmartins/micro-biblioteca/blob/main/services/inventory/products.json) es un archivo JSON que contiene la descripción de los libros en venta en la biblioteca.

-   `callback` es una función que debe ser invocada con dos parámetros:
    -   El primer parámetro es un objeto de error, caso esto ocurra. En nuestro ejemplo ningún error será retornado, por lo tanto es `null`.
    -   El segundo parámetro es el resultado de la función, en nuestro caso un `ProductResponse`, asi como fue definido en el archivo [proto/inventory.proto](https://github.com/dsanmartins/micro-biblioteca/blob/main/proto/inventory.proto).

#### Passo 4

Para finalizar, tenemos que incluir la función `SearchProductByID` en nuestro `Controller`. Para esto, usted debe incluir una nueva ruta `/product/{id}` que reciberá el ID del producto como parámetro. En la definición de la  ruta, usted debe también incluir la llamada para el método definido en el Paso 3.

Siendo mas específico, el seguinte código debe ser adicionado en la linea 44 del archivo [services/controller/index.js](https://github.com/dsanmartins/micro-biblioteca/blob/main/services/controller/index.js), luego después la ruta `/shipping/:cep`.

```js
app.get('/product/:id', (req, res, next) => {
    // Llama método del microservicio.
    inventory.SearchProductByID({ id: req.params.id }, (err, product) => {
        // Si ocurre algún error de comunicación
        // con el microservicio, retorna para el navegador.
        if (err) {
            console.error(err);
            res.status(500).send({ error: 'something failed :(' });
        } else {
            // En caso contrario, retorna el resultado del
            // microservicio (un arquivo JSON) con los datos
            // del produto buscado
            res.json(product);
        }
    });
});
```

Finalize, efectuando una llamada en el nuevo endpoint de la API: http://localhost:3000/product/1

Para clarificar: has aquí, solo implementamod la nueva operación en el back-end. Su incorporación en el frontend quedará pendiente, puesto que requiere cambiar la interfaz Web, para, por ejemplo, incluir un botón "Buscar Libro".

**IMPORTANTE**: Si todo funcionó correctamente, de un **COMMIT & PUSH** (y certifique que su repositorio en GitHub fue actualizado; esto es fundamental para que su trabajo sea devidamente corregido).

```bash
git add --all
git commit -m "Tarea práctica #1 - Microservicios"
git push origin main
```

## Tarea Práctica #2: Creando un Container Docker

(Pendiente para una próxima clase)
