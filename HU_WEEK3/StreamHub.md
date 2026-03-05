# STREAMHUB MongoDB

## Crear o Usar la base de datos.

``` 
use('StreamHub');
```

## DISENO E INSERCION DE DATOS.

### Insercion en coleccion Contenido
``` 
    db.contenido.insertMany([
  {
    title: "Interstellar",
    type: "Pelicula",
    genres: ["Ciencia Ficcion"],
    duration: 170,
    release_year: 2014,
    rating: 8.6
  },
  {
    title: "Stranger Things",
    type: "series",
    seasons: 4,
    genres: ["Horror", "Fantasia"],
    duration: 50,
    release_year: 2016,
    rating: 8.7
  },
  {
    title: "Toy Story",
    type: "movie",
    genres: ["Animacion", "Ninos"],
    duration: 81,
    release_year: 1995,
    rating: 8.3
  },
  {
    title: "Breaking Bad",
    type: "Pelicula",
    seasons: 5,
    genres: ["Crimen"],
    duration: 100,
    release_year: 2019,
    rating: 10
  }
]);
```

### Insercion en coleccion Usuarios.

``` 
db.usuarios.insertMany([
  {
    username: "kevincito",
    email: "kevin@kevin.com",
    subscription: "Premium",
    favorites_count: 12,
    watch_history: [ { title: "Interstellar", date: new Date() } ]
  },
  {
    username: "valen",
    email: "valen@valen.com",
    subscription: "Basic",
    favorites_count: 3,
    watch_history: [ { title: "Breaking Bad", date: new Date() } ]
  }
]);
```

## CONSULTAS CON OPERADORES

1. Películas con duración > 120 min ($gt)
```
db.contenido.find({ type: "movie", duration: { $gt: 120 } });
```

2. Usuarios que tienen más de 5 favoritos ($gt)
```
db.usuarios.find({ favorites_count: { $gt: 5 } });
```

3. Contenido que sea de género Ciencia Ficcion o Horror ($in)
```
db.contenido.find({ genres: { $in: ["Ciencia Ficcion", "Horror"] } });
```

4. Películas de 2010 Y con rating mayor a 8 ($and, $eq, $gt)
```
db.contenido.find({ 
  $and: [ 
    { release_year: { $eq: 2010 } }, 
    { rating: { $gt: 8.0 } } 
  ] 
});
```

5. Buscar títulos que contengan la palabra "Star" ($regex)
```
db.content.find({ title: { $regex: /Star/i } });
```


## ACTUALIZACIONES Y ELIMINACIONES

1. Actualizar el rating de Interstellar (updateOne)
```
db.content.updateOne(
  { title: "Interstellar" },
  { $set: { rating: 10 } }
);
```

2. Agregar un tag a todas las películas de Ciencia Ficcion (updateMany)
```
db.content.updateMany(
  { genres: "Ciencia Ficcion" },
  { $push: { tags: "Mejor Calificado Ciencia Ficcion" } }
);
```


3. Eliminar un usuario específico (deleteOne)
```
db.users.deleteOne({ username: "ana_cine" });
```
## INDICES PARA PERFORMANCE

1. Creamos índice en 'title' para búsquedas rápidas por nombre
```
db.content.createIndex({ title: 1 });
```

2. Índice compuesto para filtrar por género y rating (orden descendente)
```
db.content.createIndex({ genres: 1, rating: -1 });
```

3. Verificación de índices creados
```
db.content.getIndexes();
```

## PIPELINES

1. Promedio de rating por cada género
```
db.content.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", promedio: { $avg: "$rating" }, total: { $sum: 1 } } },
  { $sort: { promedio: -1 } }
]);
```

2 Reporte de películas de larga duración (>150 min)
```
db.content.aggregate([
  { $match: { type: "movie", duration: { $gt: 150 } } },
  { $project: { _id: 0, title: 1, duration: 1, rating: 1 } },
  { $sort: { duration: 1 } }
]);
```