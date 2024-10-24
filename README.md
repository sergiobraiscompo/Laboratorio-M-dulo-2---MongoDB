# Pregunta.
¿Si montaras un sitio real, ¿Qué posibles problemas pontenciales les ves a como está almacenada la información?
Indica aquí que problemas ves:
```
    - Los ID de los documentos están guardados como strings en lugar de objectid, los IDs en string hexadecimal pesan 24bytes, a diferencia de los 12 que pesa el object id.
```



# Obligatorio
# Consultas
##### Saca en una consulta cuantos alojamientos hay en España.
```
    db.listingsAndReviews.find({"address.country": "Spain"}, {name: 1, "address.country": 1});
```

Lista los 10 primeros:
Ordenados por precio de forma ascendente.
Sólo muestra: nombre, precio, camas y la localidad (address.market).
```
    
```


# Filtrando
Queremos viajar cómodos, somos 4 personas y queremos:
4 camas.
Dos cuartos de baño o más.
Sólo muestra: nombre, precio, camas y baños.
```

```

Aunque estamos de viaje no queremos estar desconectados, así que necesitamos que el alojamiento
también tenga conexión wifi. A los requisitos anteriores, hay que añadir que el alojamiento tenga wifi.
Sólo muestra: nombre, precio, camas, baños y servicios (amenities).
```

```

Y bueno, un amigo trae a su perro, así que tenemos que buscar alojamientos que permitan mascota
(Pets allowed).
Sólo muestra: nombre, precio, camas, baños y servicios (amenities).
```

```

Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen. Pero queremos que el precio nos
salga baratito (50 $), y que tenga buen rating de reviews (campo
review_scores.review_scores_rating igual o superior a 88).
Sólo muestra: nombre, precio, camas, baños, rating y localidad.
```

```

También queremos que el huésped sea un superhost (host.host_is_superhost) y que no tengamos
que pagar depósito de seguridad (security_deposit).
Sólo muestra: nombre, precio, camas, baños, rating, si el huésped es superhost, depósito de
seguridad y localidad.
```

```



# Agregaciones
Queremos mostrar los alojamientos que hay en España, con los siguientes campos:
Nombre.
Localidad (no queremos mostrar un objeto, sólo el string con la localidad).
Precio
```
    db.listingsAndReviews.aggregate([
        { $match: { "address.country": "Spain" }},
        { $project: { _id:0, name:1, location:"$address.market", price: 1 } }
    ])
```

Queremos saber cuantos alojamientos hay disponibles por pais.
```
    db.listingsAndReviews.aggregate(
        {
            $group: 
            {  
                _id: "$address.country",
                totalAccomodations: {$sum: 1}
            }
        }
    )
```



# Opcional
Queremos saber el precio medio de alquiler de airbnb en España.
```

```

¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?
```

```

Repite los mismos pasos pero agrupando también por numero de habitaciones.
```

```



# Desafio
Queremos mostrar el top 5 de alojamientos más caros en España, con los siguentes campos:
Nombre.
Precio.
Número de habitaciones
Número de camas
Número de baños
Ciudad.
Servicios, pero en vez de un array, un string con todos los servicios incluidos.
```

```
