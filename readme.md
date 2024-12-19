# Laboratorio MongoDB

Vamos a trabajar con el set de datos de Mongo Atlas _airbnb_. Lo puedes encontrar en este enlace: https://drive.google.com/drive/folders/1gAtZZdrBKiKioJSZwnShXskaKk6H_gCJ?usp=sharing

Para restaurarlo puede seguir las instrucciones de este videopost:
https://www.lemoncode.tv/curso/docker-y-mongodb/leccion/restaurando-backup-mongodb

> Acuerdate de mirar si en el directorio `/opt/app` del contenedor Mongo hay contenido de backups previos que haya que borrar

Para entregar las soluciones, añade un README.md a tu repositorio del bootcamp incluyendo enunciado y consulta (lo que pone '_Pega aquí tu consulta_').

## Introducción

En este base de datos puedes encontrar un montón de alojamientos y sus reviews, esto está sacado de hacer webscrapping.

**Pregunta**. Si montaras un sitio real, ¿Qué posibles problemas pontenciales les ves a como está almacenada la información?

```md
- Los object id de los alojamientos deberían estar en el formato objectID. Esto es debido a que pesa solo 12 bytes, además que almacena incluso la fecha de creación del objeto.
```

## Obligatorio

Esta es la parte mínima que tendrás que entregar para superar este laboratorio.

### Consultas

- Saca en una consulta cuantos alojamientos hay en España.

```js
db.listingsAndReviews.count({ "address.country": "Spain" });
```

- Lista los 10 primeros:
  - Ordenados por precio de forma ascendente.
  - Sólo muestra: nombre, precio, camas y la localidad (`address.market`).

```js
use('airbnb')

db.listingsAndReviews.find(
    { "address.country": "Spain" },
    { name: 1, price: 1, beds: 1, location: "$address.market" }
).limit(10).sort({ price: 1 })
```

### Filtrando

- Queremos viajar cómodos, somos 4 personas y queremos:
  - 4 camas.
  - Dos cuartos de baño o más.
  - Sólo muestra: nombre, precio, camas y baños.

```js
use('airbnb')

db.listingsAndReviews.find(
    { $and: [{ beds: 4 }, { bathrooms: { $gte: 2 } }] },
    { _id: 0, name: 1, price: 1, beds: 1, bathrooms: 1 },
)
```

- Aunque estamos de viaje no queremos estar desconectados, así que necesitamos que el alojamiento también tenga conexión wifi. A los requisitos anteriores, hay que añadir que el alojamiento tenga wifi.
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
use('airbnb')

db.listingsAndReviews.find(
    {
        $and: [
            { beds: 4 },
            { bathrooms: { $gte: 2 } },
            { amenities: /Wifi/ }
        ]
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        amenities: 1
    },
)
```

- Y bueno, un amigo trae a su perro, así que tenemos que buscar alojamientos que permitan mascota (_Pets allowed_).
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
use('airbnb')

db.listingsAndReviews.find(
    {
        $and: [
            { beds: 4 },
            { bathrooms: { $gte: 2 } },
            { amenities: /Wifi/ && /Pets allowed/ }
        ]
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        amenities: 1
    },
)

```

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen. Pero queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews (campo `review_scores.review_scores_rating` igual o superior a 88).
  - Sólo muestra: nombre, precio, camas, baños, rating y localidad.

```js
use('airbnb')

db.listingsAndReviews.find(
    {
        $and: [
            { price: { $lte: 50 } },
            { $or: [{ "address.market": "Barcelona" }, { "address.country": "Portugal" }] },
            { "review_scores.review_scores_rating": { $gte: 88 } },
            { amenities: /Wifi/ && /Pets allowed/ },
            { beds: { $gte: 4 } },
            { bathrooms: { $gte: 2 } },
        ]
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        rating: "$review_scores.review_scores_rating",
        location: "$address.market"
    },
)
```


- También queremos que el huésped sea un superhost (`host.host_is_superhost`) y que no tengamos que pagar depósito de seguridad (`security_deposit`).
  - Sólo muestra: nombre, precio, camas, baños, rating, si el huésped es superhost, depósito de seguridad y localidad.

```js
use('airbnb')

db.listingsAndReviews.find(
    {
        $and: [
            { price: { $lte: 50 } },
            { $or: [{ "address.market": "Barcelona" }, { "address.country": "Portugal" }] },
            { "review_scores.review_scores_rating": { $gte: 88 } },
            { amenities: /Wifi/ && /Pets allowed/ },
            { beds: { $gte: 4 } },
            { bathrooms: { $gte: 2 } },
            {
                $and: [
                    { "host.host_is_superhost": true },
                    { security_deposit: 0 }
                ]
            }
        ]
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        rating: "$review_scores.review_scores_rating",
        isSuperhost: "$host.host_is_superhost",
        security_deposit: 1,
        location: "$address.market",
    },
)
```

### Agregaciones

- Queremos mostrar los alojamientos que hay en España, con los siguientes campos:
  - Nombre.
  - Localidad (no queremos mostrar un objeto, sólo el string con la localidad).
  - Precio

```js
use('airbnb')

db.listingsAndReviews.aggregate([
    { $match: { "address.country": "Spain" } },
    { $project: { _id: 0, name: 1, location: "$address.market", price: 1 } }
])
```

- Queremos saber cuantos alojamientos hay disponibles por pais.

```js
use('airbnb')

db.listingsAndReviews.aggregate(
    {
        $group:
        {
            _id: "$address.country",
            totalAccomodations: { $sum: 1 }
        }
    }
)
```

## Opcional

- Queremos saber el precio medio de alquiler de airbnb en España.

```js
use('airbnb')

db.listingsAndReviews.aggregate([
    { $match: { "address.country": "Spain"}},
    { 
        $group: {
            _id: null,
            averageAccomodationPrice: { $avg: "$price"}
        }
    },
])
```

- ¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?

```js
use('airbnb')
// Indice temporal creado para una consulta más rápida
db.listingsAndReviews.createIndex({country: 1}, {expireAfterSeconds: 10})

db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: "$address.country",
            averageAccomodationPrice: { $avg: "$price" }
        }
    },
])

```

- Repite los mismos pasos pero agrupando también por numero de habitaciones.

```js
use('airbnb')

db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: {
                country: "$address.country",
                bedroomsNumber: "$bedrooms"
            },
            averageAccomodationPrice: { $avg: "$price" },
        }
    },
    {
        $project: {
            _id: 0,
            country: "$_id.country",
            bedrooms: "$_id.bedroomsNumber",
            averageAccomodationPrice: 1
        }
    }
])
```

## Desafio

Queremos mostrar el top 5 de alojamientos más caros en España, con los siguentes campos:

- Nombre.
- Precio.
- Número de habitaciones
- Número de camas
- Número de baños
- Ciudad.
- Servicios, pero en vez de un array, un string con todos los servicios incluidos.

```js
use('airbnb')

db.listingsAndReviews.aggregate([
    { $match: {"address.country": "Spain"} },
    { $sort: {price: -1}},
    { $limit: 5 },
    {
        $project: {
            _id: 0,
            name: 1,
            price: 1,
            bedroomNumber: "$bedrooms",
            bedNumber: "$beds",
            bathroomNumber: "$bathrooms",
            city: "$address.market",
            amenities: "$amenities".toString(),
        }
    }
])
```
