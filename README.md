# Guía para empezar a utilizar Mongoose

1- Instalar mongoose [npm](https://www.npmjs.com/package/mongoose)

```
npm i mongoose
```

2- Crear una conexión a mongoose en `configs/db.js`

En este fichero estamos conectando el nuestro servidor express con la BBDD de Mongo.
Al método `connect()` hay que pasarle como primer parametro la URL de la BBDD donde la ultima parte del path, en este coso `/library` es el nombre de la BBDD.

```js
const mongoose = require("mongoose");

mongoose
  .connect("mongodb://localhost/library", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useFindAndModify: false,
  useCreateIndex: true
})
  .then(() => console.info("> succesfully connected to mongoDB"))
  .catch((error) => {
    console.error("> error trying to connect to mongoDB: ", error.message);
    process.exit(0);
  });
```

3- Inicializar Schema in `models/books.js`

Para poder efectuar peticiones a la nuestra colección en Mongo tenemos que crear un Modelo de mongoose. Para hacer esto tenemos que importar desde mongoose la Clase `Schema`, luego creamos el modelo con `new Schema()`, el parámetro que le pasamos e la nueva instancia es el modelo del nuestro documento, que en este caso va a ser un libro con titulo, autor, año y isbn. 

```js
const mongoose = require("mongoose");
const { Schema } = mongoose;

const BookSchema = new Schema({
  title: {
    type: String,
    unique: true,
    required: true,
  },
  author: {
    type: String,
    required: true,
  },
  year: {
    type: Number,
  },
  isbn: {
    type: String,
    unique: true,
    required: true,
  },
});

const Books = mongoose.model("Books", BookSchema);

module.exports = Books;
```

4- Requerir mongoose in `index.js`

```js
require("./configs/db");
```

No es obligatorio poner exports para import con [require](https://nodejs.org/es/knowledge/getting-started/what-is-require/) una parte de código. [Stack Overflow](https://stackoverflow.com/a/38172616/9095807)

5- Crear rutas in `routes/books/index.js`

Un objeto `router` es una instancia aislada de middleware y rutas. Puede pensar en ella como una “miniaplicación”, capaz solo de realizar funciones de middleware y enrutamiento. Cada aplicación Express tiene un `router` de aplicaciones incorporado.

Un enrutador se comporta como el middleware en sí mismo, por lo que puede usarlo como argumento para `app.use()` o como argumento para el método `use()` de otro enrutador.

```js
const router = require("express").Router();

router.get("/", async (req, res, next) => {});
router.post("/", async (req, res, next) => {});
router.put("/", async (req, res, next) => {});
router.delete("/", async (req, res, next) => {});

module.exports = router;
```

6- Importar `BooksModel` para poder efectuar consultas a la BBDD

```js
const BooksModel = require("../../models/Book");
```

7- Query para devolver todos los libros

```js
const result = await BooksModel.find({}, { _id: 0, __v: 0 });
```

8- Query para crear un libro

```js
const result = await BooksModel.create(newBook);
```

9- Query para actualizar un libro

```js
const result = await BooksModel.findOneAndUpdate(
  { isbn }, // el campo por el que busca para encontrar el documento
  { year }, // los campos que actualiza
  { new: true } // para devolver el documento actualizado o el de antes de actualizar
);
```

10- Query para borrar un libro

```js
const result = await BooksModel.findOneAndDelete({ isbn });
```

11- Crear relaciones entre colecciones
1. Crear una propriedad de tipo array que contenga `Schema.Types.ObjectId` [what-is-a-schematype](https://mongoosejs.com/docs/schematypes.html#what-is-a-schematype)
```js
books: [{type: Schema.Types.ObjectId, ref: 'Books'}]
```
2. Crear una ruta para poder añadir una librería en los favoritos, para hacer eso tenemos que hacer un push del `bookId` en el array que hemos creado antes en el modelo
```js
router.get("/add-book-to-library/:name/:id", log,async (req, res) => {
  const { name, id} = req.params;
  const library = await LibraryModel.findOne({ name })
  library.books.push(id);
  await library.save();

  res.send('done');
});
```
12- Crear Middleware

1. Crear un carpeta `Middleware`
2. Crear un Middleware para que solo el admin pueda ver sus libros 
