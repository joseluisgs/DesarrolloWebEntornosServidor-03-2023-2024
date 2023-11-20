- [Introducción al testing con Jest y Supertest](#introducción-al-testing-con-jest-y-supertest)
  - [Jest](#jest)
- [Introducción a las Pruebas de Unidad en JavaScript con Jest](#introducción-a-las-pruebas-de-unidad-en-javascript-con-jest)
  - [¿Qué es Jest?](#qué-es-jest)
  - [Estructura de las Pruebas](#estructura-de-las-pruebas)
  - [Aserciones](#aserciones)
  - [Inicialización y Finalización](#inicialización-y-finalización)
  - [Tests de código asíncrono](#tests-de-código-asíncrono)
  - [Mocking](#mocking)
    - [Mockeando objetos para testear dependencias.](#mockeando-objetos-para-testear-dependencias)
  - [Supertest](#supertest)
- [Testeado en Netsjs](#testeado-en-netsjs)
  - [Testeado los servicios o providers](#testeado-los-servicios-o-providers)
  - [Testeando el controlador](#testeando-el-controlador)
    - [Probando el controlador con Supertest](#probando-el-controlador-con-supertest)
- [Práctica de clase: Testing](#práctica-de-clase-testing)
- [Proyecto](#proyecto)

![](images/06-banner.jpg)

# Introducción al testing con Jest y Supertest

## Jest
Estoy asumiendo que quieres aprender sobre el testeo en JavaScript, específicamente usando las bibliotecas Jest o Mocha/Chai. Aquí te preparé un sencillo tutorial.

# Introducción a las Pruebas de Unidad en JavaScript con Jest

## ¿Qué es Jest?

[Jest](https://jestjs.io/es-ES/) es una biblioteca de JavaScript desarrollada por Facebook para llevar a cabo pruebas de unidad y de integración en proyectos de JavaScript. Jest es bien conocido por su facilidad de configuración y por las características que ofrece, como la generación de snapshots, pruebas de instantáneas, entre otros.

## Estructura de las Pruebas

Una prueba típica con Jest implica el uso de `describe` y `it`.

**describe:** Este es un método de Jest, que se utiliza principalmente para agrupar pruebas relacionadas. Cada descripción puede contener uno o más `it`.

```javascript
describe('Grupo de pruebas', () => {
  ...

}
```

**it:** Este es otro método de Jest que se emplea para una prueba individual. Cada `it` debe estar contenido en un `describe`, aunque Jest permite pruebas `it` independientes.

```javascript
describe('Grupo de pruebas', () => {
  it('debería hacer esto', () => {
    ...
  });

  it('debería hacer aquello', () => {
    ...
  });
});
```

## Aserciones
Las aserciones, también conocidas como "matchers" en el dominio de las pruebas, son funciones que permiten comparar el valor resultante de una operación con un valor esperado. Jest proporciona una [amplia variedad de matchers](https://jestjs.io/es-ES/docs/using-matchers) para cubrir cualquier caso de prueba necesario.

Veamos algunos ejemplos de los matchers que proporciona Jest:

**1. toBe**

Comprueba la igualdad estricta (===). Es apropiado para comparar valores primitivos.

```javascript
test('Comparación con toBe', () => {
  expect(2 + 2).toBe(4);
});
```

**2. toEqual**

Comprueba la igualdad de valor y estructura. Es apropiado para comparar arrays u objetos.

```javascript
test('Comparación con toEqual', () => {
  const data = {uno: 1, dos: 2};
  expect(data).toEqual({uno: 1, dos: 2});
});
```

**3. toBeNull, toBeUndefined, toBeDefined, toBeTruthy, toBeFalsy**

Estos matchers son útiles para trabajar con valores booleanos o nulos.

```javascript
test('isNull', () => {
  var n = null;
  expect(n).toBeNull();
});

test('isUndefined', () => {
  var z;
  expect(z).toBeUndefined();
});

test('isDefined', () => {
  var z = 3;
  expect(z).toBeDefined();
});

test('isTruthy', () => {
  var truthy = "algo que no es falsy";
  expect(truthy).toBeTruthy();
});

test('isFalsy', () => {
  var falsy = 0;
  expect(falsy).toBeFalsy();
});
```

**4. toBeGreaterThan, toBeGreaterThanOrEqual, toBeLessThan, toBeLessThanOrEqual**

Estos son para comparaciones numéricas.

```javascript
test('Comparación mayor/menor que', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);
});
```

**5. toMatch**

Es útil para comparar cadenas de texto (incluso usando expresiones regulares).

```javascript
test('string matching', () => {
  expect('equipo').toMatch(/quipo/);
});
```

**6. toContain**

Es útil para comprobar si un elemento se encuentra en una lista (Array).

```javascript
test('Comprobando elementos en array', () => {
  const listaCompras = [
    'leche',
    'queso',
    'mantequilla'
  ];

  expect(listaCompras).toContain('queso');
});
```

**7. toThrow**

Este matcher es útil si quieres comprobar que una función específica lanza un error.

```javascript
function compileAndroidCode() {
  throw new Error('¡Error de compilación!');
}

test('Compilar Android lanza un error', () => {
  expect(() => compileAndroidCode()).toThrow();
});
```

Estos son solo algunos ejemplos de los "matchers" o aserciones disponibles en Jest. Puedes encontrar una lista completa de todos los matchers disponibles en la [documentación oficial de Jest](https://jestjs.io/docs/expect).

## Inicialización y Finalización
En ocasiones, las pruebas requieren preparación previa de estado o limpieza posterior para poder ejecutarse correctamente. Jest proporciona [varios métodos](https://jestjs.io/es-ES/docs/setup-teardown) para ayudarte con esto:

- `beforeAll`: Este método se ejecuta una vez antes de todas las pruebas que componen un bloque `describe`. Es útil cuando tienes una operación costosa que no necesita ser ejecutada antes de cada prueba individual, sino solamente una vez antes de todas.
- `afterAll`: Este método se ejecuta una vez después de todas las pruebas de un bloque `describe`, cuando todas las pruebas han concluido. Es útil para la limpieza de recursos que fueron inicializados en `beforeAll`.
- `beforeEach`: Este método se ejecuta antes de cada prueba dentro de un bloque `describe`. Es útil para inicializar el estado antes de cada prueba.
- `afterEach`: Este método se ejecuta después de cada prueba dentro de un bloque `describe`. Es útil para la limpieza de cualquier alteración del estado realizada durante la ejecución de una prueba.

Aquí tienes un ejemplo en código que ilustra cómo funcionan estos métodos:

```javascript
// Mock de base de datos
let database = [];

describe('Pruebas con inicialización y finalización', () => {
  beforeAll(() => {
    // Este código se ejecutará una vez antes de todas las pruebas
    database = ['usuario1', 'usuario2', 'usuario3'];
  });

  afterAll(() => {
    // Este código se ejecutará una vez después de todas las pruebas
    database = []; // limpiamos la base de datos
  });

  beforeEach(() => {
    // Este código se ejecutará antes de cada prueba. 
    database.push('usuario4');
  });

  afterEach(() => {
    // Este código se ejecutará después de cada prueba.
    database.pop();
  });

  it('debería contener usuarios', () => {
    expect(database).toContain('usuario1');
    expect(database).toContain('usuario4');
  });

  // Otros tests…
});
```

El código anterior crea una base de datos ficticia con tres usuarios. Antes de cada prueba, añade un cuarto usuario, y después de cada prueba, elimina ese cuarto usuario de la base de datos. Finalmente, después de que todas las pruebas han sido realizadas, se restaura la base de datos a su estado inicial (vacia).

## Tests de código asíncrono
Los tests asíncronos pueden ser un poco más complicados debido a que necesitas asegurarte de que Jest espera el resultado de tu prueba antes de continuar. Aquí están [algunas formas en las que puedes manejar el código asíncrono](https://jestjs.io/es-ES/docs/asynchronous) en tus tests con Jest:

**1. Devolver una Promesa**

Si devuelves una Promesa desde tu test, Jest automáticamente esperará que esa promesa se resuelva. Si la promesa es rechazada, la prueba fallará.

```javascript
it('la promesa debería ser resuelta con éxito', () => {
  return promiseFunc().then(data => {
    expect(data).toBe('hola');
  });
});
```

**2. Usar async/await**

Puedes usar async/await para escribir pruebas más legibles cuando estás trabajando con código asíncrono (te lo recomiendo).

```javascript
it('debería esperar un valor', async () => {
  const data = await asyncFunc();
  expect(data).toBe('hola');
});
```

**3. Usar el argumento 'done'**

Jest provee un argumento `done` en el callback del test. Puedes llamar a esta función cuando tu código asíncrono ha completado, de esta manera le indicas a Jest que tu test ha terminado. Si no llamas a `done` dentro del test, Jest asumirá que el test ha terminado apenas termine de ejecutarse la función de pruebas síncrona y continuará con los siguientes tests.

```javascript
it('callback ha sido llamado', done => {
  function callback(data) {
    try {
      expect(data).toBe('hola');
      done();
    } catch (error) {
      done(error);  // Esto hace fallar la prueba si hay error
    }
  }

  callbackFunc(callback);
});
```

Recuerda que tienes que manejar los errores en tu código asíncrono correctamente, de otra manera, los errores podrían no ser capturados por Jest y tus pruebas podrían pasar cuando en realidad deberían fallar. Así que siempre asegúrate de usar `.catch` en tus promesas o try/catch al usar async/await.

## Mocking

Puedes simular funciones y módulos para aislar el código de prueba de otras partes del sistema. Esto hace que las pruebas unitarias sean realmente "unitarias". Aquí te muestro cómo hcacerlo con [Jest](https://jestjs.io/es-ES/docs/mock-functions).

**Mock de una función:**
```javascript
const mockFunc = jest.fn(); // crea una función simulada
mockFunc.mockReturnValue(42); // hace que la función simulada devuelva 42
console.log(mockFunc()); // 42 
```

**Mock de un módulo:**

Imagina que tienes un módulo llamado `suma` que quieres simular.

```javascript
// __mocks__/suma.js
module.exports = () => 42;
```

Entonces, puedes usar `jest.mock` para simularlo.

```javascript
jest.mock('./suma');
```

**Mock de métodos individuales:**

Imagina que tienes una clase con varios métodos. Quieres simular solo uno de ellos.

```javascript
const math = require('./math');
math.add = jest.fn(() => 42); // mockea la función
math.subtract = jest.fn(() => 8); // mockea la función
```

Esto es solo el comienzo de lo que puedes hacer con Jest. Te recomendaría leer más acerca de las pruebas en general y las funcionalidades que Jest tiene para ofrecer. También hay otros frameworks y librerías de pruebas en JavaScript, como Mocha y Jasmine, que podrías considerar explorar para entender mejor cómo funciona el testeo en JavaScript.

### Mockeando objetos para testear dependencias.
Las burlas (mocks) son una técnica esencial en las pruebas unitarias y Jest proporciona numerosas utilidades para controlar el comportamiento de las funciones y verificar cómo se utilizan. Aquí te mostraré cómo puedes mockear los métodos de objeto, comprobar si se han llamado y cómo usar `.mockReturnThis` y `.mockReturnOnce`.

**1. Mockear los métodos de un objeto**

Puedes crear un objeto mock con Jest utilizando `jest.mock()` y luego definir el comportamiento de las funciones individuales mediante `.mockImplementation()` o simplemente devolviendo un valor específico con `.mockReturnValue`.

Por ejemplo, si tienes un objeto `repositorio` que tiene un método `obtenerUsuarios`, puedes mockear este método de la siguiente manera:

```javascript
const repositorio = {
  obtenerUsuarios: jest.fn()
};

repositorio.obtenerUsuarios.mockReturnValue(Promise.resolve(['usuario1', 'usuario2']));
```

**2. Comprobar si un método mock se ha llamado**

Una vez que has creado tu mock, puedes comprobar si se ha llamado utilizando `.toHaveBeenCalled()` y cuántas veces se ha llamado con `.toHaveBeenCalledTimes(n)`.

```javascript
// Imagina que tu función de servicio llama a repositorio.obtenerUsuarios
await servicio.funcion();

expect(repositorio.obtenerUsuarios).toHaveBeenCalled();
expect(repositorio.obtenerUsuarios).toHaveBeenCalledTimes(1);
```

**3. Diferencia entre .mockReturnThis() y .mockReturnOnce()**

`.mockReturnThis()` es un método en un objeto mock de Jest que hace que la función mock devuelva `this`, es útil cuando estás testeando una cadena de métodos de un objeto.

`.mockReturnOnce()` es otro método en un objeto mock de Jest que hace que la función devuelva un valor específico (que le pases como argumento) la próxima vez que se llame.

Por ejemplo:

```javascript
const mockFunc = jest.fn()
  .mockReturnOnce('hello')   // Primera llamada
  .mockReturnOnce('world')   // Segunda llamada
  .mockReturnThis();         // Todas las demás llamadas

// Usando la función mock
console.log(mockFunc());     // 'hello'
console.log(mockFunc());     // 'world'
console.log(mockFunc());     // mockFunc {}
```

En el caso de tu repositorio, si quisieras que los métodos de un objeto se comporten de manera diferente en cada llamada, podrías usar algo similar a esto:

```javascript
// Mock del comportamiento del método y además este puede ser asíncrono, por eso las promesas!!
repositorio.obtenerUsuarios
  .mockReturnValueOnce(Promise.resolve(['usuario1']))
  .mockReturnValueOnce(Promise.resolve(['usuario2']))
  .mockReturnValueOnce(Promise.resolve(['usuario3']));

// Llamada del servicio
const usuarios1 = await servicio.funcion();
const usuarios2 = await servicio.funcion();
const usuarios3 = await servicio.funcion();

// Verificación del comportamiento esperado
expect(usuarios1).toEqual(['usuario1']);
expect(usuarios2).toEqual(['usuario2']);
expect(usuarios3).toEqual(['usuario3']);
```

Por último, si te interesa comprobar los argumentos con los que se llama a una función mock, puedes utilizar `.toHaveBeenCalledWith(arg1, arg2, ...)`. Este método te permite verificar que la función mock se llamó con los argumentos específicos al menos una vez.

```javascript
service.funcion('test');

expect(repositorio.obtenerUsuarios).toHaveBeenCalledWith('test');
```

## Supertest
[Supertest](https://github.com/ladjs/supertest) es una biblioteca extremadamente útil para realizar pruebas de API HTTP en Node.js usando la biblioteca de pruebas Mocha o Jest. Se basa en el módulo Superagent para proporcionar una interfaz de alto nivel para probar las respuestas HTTP.

```javascript
describe('Tests del API', () => {
  // La petición GET
  it('GET / debería responder con hola mundo', async () => {
    const res = await request(app).get('/');
    expect(res.text).toEqual('hola mundo');
    expect(res.status).toEqual(200);
  });

  // La petición POST
  it('POST / debería responder con 201', async () => {
    const data = { name: 'John Doe', email: 'john.doe@example.com' };
    const res = await request(app).post('/').send(data);
    expect(res.status).toEqual(201);
    // Puedes hacer más aserciones sobre res.body si deseas
  });

  // La petición PUT
  it('PUT /:id debería responder con 200', async () => {
    const data = { name: 'Jane Doe', email: 'jane.doe@example.com' };
    const res = await request(app).put('/1').send(data);
    expect(res.status).toEqual(200);
    // Puedes hacer más aserciones sobre res.body si deseas
  });

  // La petición DELETE
  it('DELETE /:id debería responder con 200', async () => {
    const res = await request(app).delete('/1');
    expect(res.status).toEqual(200);
  });
});
```

# Testeado en Netsjs
Nestjs viene con amplias utiliades para reaizar test adaptado a su forma de crear los componentes y módulos. Puedes ver más en su [documentación](https://docs.nestjs.com/fundamentals/testing).

## Testeado los servicios o providers
- beforeEach: Esto se llama antes de cada prueba. Se utiliza para configurar cualquier cosa que tus pruebas individuales puedan necesitar. Aquí se está creando una instancia de un módulo de prueba que contiene ciertos proveedores que tus pruebas podrían necesitar, incluyendo servicios u otros providers, CategoriasMapper y un repositorio para CategoriaEntity.
- Test.createTestingModule: Esto es específico de NestJS. Se utiliza para crear un módulo de prueba que puede tener sus propios proveedores y control inyectados. 
- getRepositoryToken: Esta es una función auxiliar de NestJS que se utiliza para obtener el token de inyección de dependencias para un repositorio TypeORM. Se usa para poder "mockear" el repositorio en el módulo de prueba.
- service = module.get<CategoriasService>(CategoriasService): Se obtiene una instancia del servicio testado. También se genera una instancia del repositorio, que se usará para hacer mock de los métodos del repositorio como find, findOneBy, save y remove.
- Otra forma: para haver el test unitario de controladores, lo que también podemos es cargar nuestro controlador y servicio en el módulo de test y luego hacer mocks, con la función spy con mockImplementation de los métodos del servicio. Usamos un describe por cada método is este tiene distintos resultados (correcto o excepciones)
```ts
 it('should return a not acceptable exception', async () => {
      const mockUSer = {
        id: 0,
        name: '',
        surname: '',
      }
      // Creamos un mock del metodo create del servicio, cuando se llame al método create del servicio
      jest.spyOn(userService, 'create').mockImplementation(() => mockUSer)
      // Llamamos al metodo create del controlador y comprbamos que se lanza una excepción
      try {
        await userController.create(mockUSer)
      } catch (e) {
        // Comprobamos que el mensaje de la excepción sea el esperado
        expect(e.message).toBe('invalid user data')
      }
    })
```

Aquí te muestro un ejemplo de las forma recomendada o más sencilla para ti ahora mientras aprendes:

```ts
describe('CategoriasService', () => {
  let mapper: CategoriasMapper
  let service: CategoriasService
  let repo: Repository<CategoriaEntity>

  beforeEach(async () => {
    const categoriasMapper = {
      convertToEntity: jest.fn(),
      convertToDto: jest.fn(),
      // También puedes agregar todas las otras funciones del mapper que puedas necesitar en tus pruebas
    };
    // Creamos un módulo de prueba de NestJS que nos permitirá crear una instancia de nuestro servicio.
    const module: TestingModule = await Test.createTestingModule({
      // Proporcionamos una lista de dependencias que se inyectarán en nuestro servicio.
      providers: [
        CategoriasService, // El servicio que lo necesito
        { provide: CategoriasMapper, useValue: categoriasMapper }, // Usamos el objeto mock que acabamos de crear
        // CategoriasMapper, // El mapeador! (también podemos mockearlo)
        {
          provide: getRepositoryToken(CategoriaEntity), // Obtenemos el token de la entidad CategoriaEntity para inyectarlo en el servicio.
          useClass: Repository, // Creamos una instancia de la clase Repository para inyectarla en el servicio
        },
      ],
    }).compile() // Compilamos el módulo de prueba.

    service = module.get<CategoriasService>(CategoriasService) // Obtenemos una instancia de nuestro servicio.
    // getRepositoryToken es una función de NestJS que se utiliza para generar un token de inyección de dependencias para un repositorio de TypeORM.
    service = module.get<CategoriasService>(CategoriasService)
    repo = module.get<Repository<CategoriaEntity>>(getRepositoryToken(CategoriaEntity))
    mapper = module.get<CategoriasMapper>(CategoriasMapper);// Asegúrate de obtener la instancia del mapper
  })

  it('should be defined', () => {
    expect(service).toBeDefined() // para ver si se ha creado...
  })

  describe('findAll', () => {
    it('should return all categories', async () => {
  
      const testCategories = [new CategoriaEntity()]
      const testCategoriaDto = new CategoriaDto()
      jest.spyOn(repo, 'find').mockResolvedValue(testCategories)
       jest.spyOn(mapper, 'convertToDto').mockReturnValue(testCategoryDto);

      // expect that the service will return all categories
      expect(await service.findAll()).toEqual([testCategoriaDto])
    })
  })

  describe('findOne', () => {
    it('should return a single category', async () => {
      const testCategory = new CategoriaEntity();
      const testCategoryDto = new CategoriaDto(); // Suponiendo que tienes una clase CategoriaDto

      jest.spyOn(repo, 'findOne').mockResolvedValue(testCategory); // Bott usamos findOne en lugar de findOneBy para más generalidad
      jest.spyOn(mapper, 'convertToDto').mockReturnValue(testCategoryDto);

      const result = await service.findOne('1');
      expect(result).toEqual(testCategoryDto);
      expect(mapper.convertToDto).toHaveBeenCalledWith(testCategory); // Verifica que el mapper se usó correctamente
    });

    it('should throw an error if the category does not exist', async () => {
      jest.spyOn(repo, 'findOneBy').mockResolvedValue(null)
      await expect(service.findOne('1')).rejects.toThrow(NotFoundException)
    })
  })

  // Resto de los test!!
})
```

## Testeando el controlador

Podemos testear el controlador siguiendo la misma idea que con el servicio y sus dependencias. Pero esta vez crearemos mocks de las funciones que vayamos a usar del servicio. Podemos crear un objeto para proveerlo y usar el mock, o definirlo sobre la marcha. Esta vez para ver otra forma alternativa lo haremos de la segunda forma:

```ts
describe('CategoriasController', () => {
  let controller: CategoriasController
  let service: CategoriasService

  beforeEach(async () => {
    // Creamos un módulo de prueba de NestJS que nos permitirá crear una instancia de nuestro controlador.
    const module: TestingModule = await Test.createTestingModule({
      controllers: [CategoriasController],
      providers: [
        {
          provide: CategoriasService,
          useValue: {
            findAll: jest.fn(),
            findOne: jest.fn(),
            create: jest.fn(),
            update: jest.fn(),
            removeSoft: jest.fn(),
          },
        },
      ],
    }).compile()

    controller = module.get<CategoriasController>(CategoriasController)
    service = module.get<CategoriasService>(CategoriasService)
  })

  it('should be defined', () => {
    expect(controller).toBeDefined()
  })

  describe('findAll', () => {
    it('should get all categorias', async () => {
      const mockResult: Array<CategoriaEntity> = []
      jest.spyOn(service, 'findAll').mockResolvedValue(mockResult)
      const result = await controller.findAll()
      expect(service.findAll).toHaveBeenCalled()
      expect(result).toBeInstanceOf(Array)
    })
  })

  describe('findOne', () => {
    it('should get one categoria', async () => {
      const id = 'uuid'
      const mockResult: CategoriaEntity = new CategoriaEntity()

      jest.spyOn(service, 'findOne').mockResolvedValue(mockResult)
      await controller.findOne(id)
      expect(service.findOne).toHaveBeenCalledWith(id)
      expect(mockResult).toBeInstanceOf(CategoriaEntity)
    })

    it('should throw NotFoundException if categoria does not exist', async () => {
      const id = 'a uuid'
      jest.spyOn(service, 'findOne').mockRejectedValue(new NotFoundException())
      await expect(controller.findOne(id)).rejects.toThrow(NotFoundException)
    })
  })

  describe('create', () => {
    it('should create a categoria', async () => {
      const dto: CreateCategoriaDto = {
        nombre: 'test',
      }
      const mockResult: CategoriaEntity = new CategoriaEntity()
      jest.spyOn(service, 'create').mockResolvedValue(mockResult)
      await controller.create(dto)
      expect(service.create).toHaveBeenCalledWith(dto)
    })
  })

  // reto de métodos
})
```
### Probando el controlador con Supertest
Podemos hacer test e2e con supertest, pero además si queremos podemos mockear las dependencias que use. De esta manera no será un test de integración al uso, porque para eso usaremos Postman, pero si nos servirá para ver si actuamos correctamente (y para practicar).

De nuevo podemos hacer un mock de servicio o servicios que usemos y sus métodos y esta vez lo vamos a subreescribir para usarlo.

Te dejo un ejemplo

```ts

// https://blog.logrocket.com/end-end-testing-nestjs-typeorm/

describe('CategoriasController (e2e)', () => {
  let app: INestApplication
  const myEndpoint = `/categorias`

  const myCategoria: CategoriaEntity = {
    id: '7958ef01-9fe0-4f19-a1d5-79c917290ddf',
    nombre: 'nombre',
    isDeleted: false,
    createdAt: new Date(),
    updatedAt: new Date(),
    productos: [],
  }

  const createCategoriaDto = {
    nombre: 'nombre',
  }

  const updateCategoriaDto = {
    nombre: 'nombre',
    isDeleted: false,
  }

  // Mock de servicio y sus metodos
  const mockCategoriasService = {
    findAll: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    remove: jest.fn(),
    removeSoft: jest.fn(),
  }

  beforeEach(async () => {
    // Cargamos solo el controlador y el servicio que vamos a probar, no el módulo que arrastra con todo
    // No es de integración si no e2e, con mocks
    const moduleFixture: TestingModule = await Test.createTestingModule({
      controllers: [CategoriasController],
      providers: [
        CategoriasService,
        { provide: CategoriasService, useValue: mockCategoriasService },
      ],
    })
      // Le decimos a Nest que inyecte nuestro mock de servicio en lugar del servicio real.
      .overrideProvider(CategoriasService)
      .useValue(mockCategoriasService)
      .compile()

    app = moduleFixture.createNestApplication()
    await app.init()
  })

  afterAll(async () => {
    await app.close()
  })

  describe('GET /categorias', () => {
    it('should return an array of categorias', async () => {
      // Configurar el mock para devolver un resultado específico
      mockCategoriasService.findAll.mockResolvedValue([myCategoria])

      const { body } = await request(app.getHttpServer())
        .get(myEndpoint)
        .expect(200)
      expect(() => {
        expect(body).toEqual([myCategoria])
        expect(mockCategoriasService.findAll).toHaveBeenCalled()
      })
    })
  })

  describe('GET /categorias/:id', () => {
    it('should return a single categoria', async () => {
      mockCategoriasService.findOne.mockResolvedValue(myCategoria)

      const { body } = await request(app.getHttpServer())
        .get(`${myEndpoint}/${myCategoria.id}`)
        .expect(200)
      expect(() => {
        expect(body).toEqual(myCategoria)
        expect(mockCategoriasService.findOne).toHaveBeenCalled()
      })
    })

    it('should throw an error if the category does not exist', async () => {
      mockCategoriasService.findOne.mockRejectedValue(new NotFoundException())

      await request(app.getHttpServer())
        .get(`${myEndpoint}/${myCategoria.id}`)
        .expect(404)
    })
  })

  describe('POST /categorias', () => {
    it('should create a new categoria', async () => {
      mockCategoriasService.create.mockResolvedValue(myCategoria)

      const { body } = await request(app.getHttpServer())
        .post(myEndpoint)
        .send(createCategoriaDto)
        .expect(201)
      expect(() => {
        expect(body).toEqual(myCategoria)
        expect(mockCategoriasService.create).toHaveBeenCalledWith(
          createCategoriaDto,
        )
      })
    })
  })
  // Resto de test
})
```

# Práctica de clase: Testing
1. Realiza los test unitarios de tus servicios, mapeadores y controladores de funkos y categorías, mockeando las dependencias necesarias.
2. Crea el test del edpoint de categorías usando supertest mockeando los elementos necesarios.

# Proyecto
Puedes consultar esta parte en [el proyecto de ejemplo](https://github.com/joseluisgs/DesarrolloWebEntornosServidor-03-Proyecto-2023-2024/releases/tag/productos_categorias).