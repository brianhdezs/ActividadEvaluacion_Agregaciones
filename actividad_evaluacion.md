# ACTIVIDAD DE EVALUACION - AGREGACIONES EN MONGO

## BRIAN EMMANUEL FLORES HERNANDEZ

### 1. Cuenta los productos de tipo “medio”, usando un método básico
  ```json
    db.productos.find({ tipo: "medio" }).count()
  ```
### 2. Indicar con un distinct, las empresas (fabricantes) que hay en la colección
```json
  db.productos.distinct("fabricante")
 ```
### 3. Usando aggregate, visualizar los productos que tengan más de 80 unidades
  db.productos.aggregate([{ $match: { unidades: { $gt: 80 } } }])
### 4. Con $project visualizar solo el nombre, unidades y precio de los productos que tengan menos de 10 unidades
 ```json
  db.productos.aggregate([{ $match: { unidades: { $lt: 10 } } }, { $project: { _id: 0, nombre: 1, unidades: 1, precio: 1 } }])
 
 ```

### 5. Con $project ponemos el fabricante, pero le cambiamos el nombre por “empresa”. Usamos el mismo comando anterior
 ```json
db.productos.aggregate([{ $match: { unidades: { $lt: 10 } } }, { $project: { _id: 0, nombre: 1, unidades: 1, precio: 1, empresa: "$fabricante" } }])
 ```

### 6. Añadir a la consulta anterior un campo calculado que se llame total y que multiplique precio por unidades.
 ```json
  db.productos.aggregate([{ $match: { unidades: { $lt: 10 } } }, { $addFields: { total: { $multiply: ["$precio", "$unidades"] } } }, { $project: { _id: 0, nombre: 1, unidades: 1, precio: 1, empresa: "$fabricante", total: 1 } }])
 ```
### 7. Hacer que el nombre salga en mayúsculas con el operador $toUpper
 ```json
db.productos.aggregate([
    { $match: { unidades: { $lt: 10 } } },
    { $addFields: { total: { $multiply: ["$precio", "$unidades"] } } },
    { $project: { _id: 0, nombre: { $toUpper: "$nombre" }, unidades: 1, precio: 1, empresa: "$fabricante", total: 1 } }
])
 ```

### 8. Añadir un campo calculado que ponga el nombre del producto y el tipo concatenado con el operador $concat. Le llamamos al campo “completo”
 ```json
db.productos.aggregate([
    { $match: { unidades: { $lt: 10 } } },
    { $addFields: { 
        total: { $multiply: ["$precio", "$unidades"] },
        completo: { $concat: ["$nombre", " - ", "$tipo"] } 
    } },
    { $project: { _id: 0, nombre: { $toUpper: "$nombre" }, unidades: 1, precio: 1, empresa: "$fabricante", total: 1, completo: 1 } }
])
 ```

### 9. Ordena el resultado por el campo “total”
 ```json
db.productos.aggregate([
    { $match: { unidades: { $lt: 10 } } },
    { $addFields: { 
        total: { $multiply: ["$precio", "$unidades"] },
        completo: { $concat: ["$nombre", " - ", "$tipo"] } 
    } },
    { $project: { _id: 0, nombre: { $toUpper: "$nombre" }, unidades: 1, precio: 1, empresa: "$fabricante", total: 1, completo: 1 } },
    { $sort: { total: 1 } }
])
 ```

### 10. Haciendo una nueva consulta, averiguar el numero de productos por tipo de producto
 ```json
db.productos.aggregate([
    { $group: { _id: "$tipo", totalProductos: { $sum: 1 } } }
])
 ```


### 11. Añadir el valor mayor y el menor
 ```json
db.productos.aggregate([
    { $group: { 
        _id: "$tipo", 
        totalProductos: { $sum: 1 }, 
        valores: { $push: "$precio" } 
    } },
    { $addFields: { 
        mayorValor: { $arrayElemAt: ["$valores", -1] }, 
        menorValor: { $arrayElemAt: ["$valores", 0] } 
    } },
    { $project: { _id: 1, totalProductos: 1, mayorValor: 1, menorValor: 1 } }
])
 ```

### 12. Añade el total de unidades por cada tipo
 ```json
db.productos.aggregate([
    { $group: { 
        _id: "$tipo", 
        totalProductos: { $sum: 1 }, 
        totalUnidades: { $sum: "$unidades" },
        valores: { $push: "$precio" } 
    } },
    { $addFields: { 
        mayorValor: { $arrayElemAt: ["$valores", -1] }, 
        menorValor: { $arrayElemAt: ["$valores", 0] } 
    } },
    { $project: { _id: 1, totalProductos: 1, totalUnidades: 1, mayorValor: 1, menorValor: 1 } }
])

 ```

### 13. Con el operador $set y el operador “$substr” visualiza todos los datos del producto "Small Metal Tuna" y los primeros 5 caracteres del nombre
 ```json
db.productos.aggregate([
    { $match: { nombre: "Small Metal Tuna" } },
    { $set: { primeros5Caracteres: { $substr: ["$nombre", 0, 5] } } },
    { $project: { _id: 0 } }
])
 ```



### 14. Creamos una salida que tenga el nombre del articulo y el total (precio por unidades) y lo guardamos en una colección denominada productos2

 ```json
db.productos.aggregate([
    { $addFields: { total: { $multiply: ["$precio", "$unidades"] } } },
    { $project: { _id: 0, nombre: 1, total: 1 } },
    { $out: "productos2" }
])
 ```

### 15. Creamos una salida que tenga el nombre del articulo y el total (precio por unidades) y lo guardamos en una colección denominada productos2
 ```json
db.productos.aggregate([
    { $addFields: { total: { $multiply: ["$precio", "$unidades"] } } },
    { $project: { _id: 0, nombre: 1, total: 1 } },
    { $out: "productos2" }
])
 ```

### 16. Comprobamos que se ha creado
 ```json
productos> db.getCollectionNames()
[ 'productos', 'productos2' ]
 ```
### 17. Hacemos un find para comprobar el resultado
 ```json
productos> db.productos2.find()
[
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc0'),
    nombre: 'Fantastic Wooden Fish',
    total: 27645
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc1'),
    nombre: 'Rustic Concrete Pants',
    total: 18414
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc2'),
    nombre: 'Small Soft Fish',
    total: 18144
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc3'),
    nombre: 'Practical Soft Pants',
    total: 2747
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc4'),
    nombre: 'Ergonomic Metal Ball',
    total: 1230
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc5'),
    nombre: 'Small Steel Gloves',
    total: 1665
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc6'),
    nombre: 'Ergonomic Wooden Shirt',
    total: 14994
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc7'),
    nombre: 'Handmade Steel Chair',
    total: 5392
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc8'),
    nombre: 'Handcrafted Soft Gloves',
    total: 4512
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbc9'),
    nombre: 'Fantastic Concrete Salad',
    total: 12985
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbca'),
    nombre: 'Handmade Plastic Hat',
    total: 1771
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbcb'),
    nombre: 'Refined Wooden Tuna',
    total: 8480
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbcc'),
    nombre: 'Refined Concrete Salad',
    total: 11610
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbcd'),
    nombre: 'Unbranded Soft Fish',
    total: 9434
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbce'),
    nombre: 'Small Concrete Fish',
    total: 5040
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbcf'),
    nombre: 'Refined Concrete Bike',
    total: 2700
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbd0'),
    nombre: 'Tasty Cotton Pants',
    total: 3536
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbd1'),
    nombre: 'Incredible Granite Gloves',
    total: 20590
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbd2'),
    nombre: 'Practical Metal Mouse',
    total: 6650
  },
  {
    _id: ObjectId('6614c824850c48bbe4bbdbd3'),
    nombre: 'Handcrafted Steel Chicken',
    total: 6215
  }
]
Type "it" for more
productos>
 ```
 ---