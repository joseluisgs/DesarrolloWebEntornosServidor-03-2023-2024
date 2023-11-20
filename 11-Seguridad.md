- [Seguridad: Autenticación, Autorización, JWT y SSL](#seguridad-autenticación-autorización-jwt-y-ssl)
  - [Instalación de dependencias](#instalación-de-dependencias)
  - [Configurando las opciones de JWT](#configurando-las-opciones-de-jwt)
  - [Implementando la autenticación y autorización](#implementando-la-autenticación-y-autorización)
    - [Módulo de autenticación](#módulo-de-autenticación)
    - [Estrategia de autenticación JWT](#estrategia-de-autenticación-jwt)
    - [Guardas de autorización y autenticación](#guardas-de-autorización-y-autenticación)
    - [Implementando el servicio de autenticación](#implementando-el-servicio-de-autenticación)
  - [Protegiendo las rutas](#protegiendo-las-rutas)
- [SSL y HTTPS](#ssl-y-https)
  - [Testeando la seguridad](#testeando-la-seguridad)
- [Práctica de clase: Seguridad](#práctica-de-clase-seguridad)
- [Proyecto](#proyecto)

![](images/11-banner.webp)

# Seguridad: Autenticación, Autorización, JWT y SSL
En este apartado veremos los conceptos de seguridad más importantes en el desarrollo de APIs REST, como son la autenticación, autorización, JWT y SSL. 

## Instalación de dependencias
Lo primero que vamos a necesitar es instalar las dependencias necesarias para trabajar con con la autenticación JWT en Nestjs. 

Podemos apostar por usar el módulo de autenticación de Nestjs, pero vamos a usar [Passport](https://www.passportjs.org/), que es un módulo de autenticación más genérico y que nos permite usar diferentes estrategias de autenticación, entre ellas JWT.

Para ello, vamos a instalar las siguientes dependencias:

- **@nestjs/jwt**: Módulo de JWT para Nestjs.
- **@nestjs/passport**: Módulo de passport para Nestjs.
- **passport**: Módulo de passport.
- **passport-jwt**: Módulo de passport para JWT.
- **bcryptjs**: Módulo para encriptar contraseñas.
- **@types/passport-jwt**: Tipado de passport para JWT.
- **@types/bcryptjs**: Tipado de bcryptjs.

```bash
npm install --save @nestjs/jwt @nestjs/passport passport passport-jwt bcryptjs 
npm install --save-dev @types/passport-jwt @types/bcryptjs
```

## Configurando las opciones de JWT
En nuestro fichero .env podemos guardar las opciones de JWT, como el secreto, el tiempo de expiración, etc. Por ejemplo:

```bash
JWT_SECRET=secret
JWT_EXPIRATION_TIME=3600
```

## Implementando la autenticación y autorización

### Módulo de autenticación

Ahora vamos a crear un módulo de autenticación, que será el encargado de gestionar la autenticación de los usuarios. Para ello, vamos a crear un módulo llamado `auth` dentro de la carpeta `src` y dentro de él, vamos a crear un fichero llamado `auth.module.ts` con el siguiente contenido:

```typescript
@Module({
  imports: [
    // Configuración edl servicio de JWT
    JwtModule.register({
      // Lo voy a poner en base64
      secret: Buffer.from(
        process.env.TOKEN_SECRET ||
          'Me_Gustan_Los_Pepinos_De_Leganes_Porque_Son_Grandes_Y_Hermosos',
        'utf-8',
      ).toString('base64'),
      signOptions: {
        expiresIn: Number(process.env.TOKEN_EXPIRES) || 3600, // Tiempo de expiracion
        algorithm: 'HS512', // Algoritmo de encriptacion
      },
    }),
    // Importamos el módulo de passport con las estrategias
    PassportModule.register({ defaultStrategy: 'jwt' }),
    // Importamos el módulo de usuarios porque usaremos su servicio
    UsersModule,
  ],
  controllers: [AuthController],
  providers: [AuthService, AuthMapper, JwtAuthStrategy],
})
export class AuthModule {}
```

Como vemos, estamos importando el módulo de JWT y configurándolo con las opciones que hemos definido en el fichero .env. También estamos importando el módulo de passport y el módulo de usuarios, ya que usaremos su servicio.

### Estrategia de autenticación JWT

Ahora vamos a crear un fichero llamado `jwt.strategy.ts` dentro de la carpeta `src/auth`. En él, vamos a crear una clase llamada `JwtAuthStrategy` que extenderá de `PassportStrategy` y que implementará la estrategia de autenticación JWT. Esta clase tendrá un constructor que recibirá el servicio de autenticación y en el método `validate` validaremos el token y devolveremos el usuario. Las estrategias de Passport deben implementar el método `validate` que recibe el payload del token y devuelve el usuario.

El código de esta clase será el siguiente:

```typescript
@Injectable()
export class JwtAuthStrategy extends PassportStrategy(Strategy) {
  constructor(private readonly authService: AuthService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(), // el token como barer token
      ignoreExpiration: false, // ignora la expiracion
      // La clave secreta
      secretOrKey: Buffer.from(
        process.env.TOKEN_SECRET ||
          'Me_Gustan_Los_Pepinos_De_Leganes_Porque_Son_Grandes_Y_Hermosos',
        'utf-8',
      ).toString('base64'),
    })
  }

  // Si se valida obtenemos el role
  async validate(payload: Usuario) {
    const id = payload.id
    return await this.authService.validateUser(id)
  }
}
```

### Guardas de autorización y autenticación
El siguiente paso es crear las guardas de autorización y autenticación. La primera de ellas será la guarda de autenticación, que será la encargada de comprobar que el usuario está autenticado. Para ello, vamos a crear un fichero llamado `jwt-auth.guard.ts` dentro de la carpeta `src/auth` con el siguiente contenido. Esta extiende de `AuthGuard`  de Passport y recibe como parámetro la estrategia de autenticación JWT que hemos creado anteriormente.

```ts
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return super.canActivate(context)
  }
}
```

Nuestro siguiente paso es crear la guarda de autorización, que será la encargada de comprobar que el usuario tiene los roles necesarios para acceder a la ruta. Para ello, vamos a crear un fichero llamado `roles-auth.guard.ts` dentro de la carpeta `src/auth` con el siguiente contenido. `RolesAuthGuard` es un guardián personalizado que implementa la interfaz `CanActivate` de NestJS. El constructor de `RolesAuthGuard` inyecta una instancia de `Reflector`, que es una utilidad proporcionada por NestJS para recuperar metadatos.

La función canActivate es el corazón del guardián. Se llama cada vez que una solicitud entra en una ruta que está protegida por este guardián.

Dentro de canActivate, primero usamos Reflector para obtener los roles requeridos del manejador de ruta. Estos roles son metadatos que se agregaron al manejador de ruta utilizando el decorador `@Roles`.

Si no se requieren roles para la ruta, permitimos que la solicitud pase.

Luego, obtenemos el objeto de usuario de la solicitud. Este objeto de usuario debe haber sido adjuntado a la solicitud por un middleware o guardián anterior, como `JwtAuthGuard`. Comprobamos si el usuario tiene alguno de los roles requeridos. Si es así, permitimos que la solicitud pase. Si no, la solicitud es denegada.

```ts
Injectable()
export class RolesAuthGuard implements CanActivate {
  private readonly logger = new Logger(RolesAuthGuard.name)

  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler())
    this.logger.log(`Roles: ${roles}`)
    if (!roles) {
      return true
    }
    const request = context.switchToHttp().getRequest()
    const user = request.user
    this.logger.log(`User roles: ${user.roles}`)
    // Al menos tenga un rol de los requeridos!!
    const hasRole = () => user.roles.some((role) => roles.includes(role))
    return user && user.roles && hasRole()
  }
}

/*
SetMetadata es una función proporcionada por NestJS que te permite agregar metadatos personalizados
a los manejadores de ruta. Estamos creando una nueva función, Roles, que toma una lista de roles
y utiliza SetMetadata para agregarlos como metadatos al manejador de ruta.
Podrás poner los roles requeridos en la ruta usando el decorador @Roles.
 */
// El decorador
export const Roles = (...roles: string[]) => SetMetadata('roles', roles)
```

### Implementando el servicio de autenticación

En nuestro servicio haremos uso de los servicios de usuarios y de JWT. Para ello, vamos a crear un fichero llamado `auth.service.ts` dentro de la carpeta `src/auth` con el siguiente contenido. En él, haremos uso de Bcrypt para encriptar las contraseñas y comparar las contraseñas encriptadas con las que nos llegan en las peticiones (todo desde otro servicio en el módulo de usuarios). También haremos uso del servicio de usuarios para crear usuarios y buscarlos por nombre de usuario. Y generaremos los tokens JWT.

```typescript
@Injectable()
export class AuthService {
  private readonly logger = new Logger(AuthService.name)

  constructor(
    private readonly usersService: UsersService,
    private readonly authMapper: AuthMapper,
    private readonly jwtService: JwtService,
  ) {}

  async singUp(userSignUpDto: UserSignUpDto) {
    this.logger.log(`singUp ${userSignUpDto.username}`)

    const user = await this.usersService.create(
      this.authMapper.toCreateDto(userSignUpDto),
    )
    return this.getAccessToken(user.id)
  }

  async singIn(userSignInDto: UserSignInDto) {
    this.logger.log(`singIn ${userSignInDto.username}`)
    const user = await this.usersService.findByUsername(userSignInDto.username)
    if (!user) {
      throw new BadRequestException('username or password are invalid')
    }
    const isValidPassword = await this.usersService.validatePassword(
      userSignInDto.password, // plain
      user.password, // hash
    )
    if (!isValidPassword) {
      throw new BadRequestException('username or password are invalid')
    }
    return this.getAccessToken(user.id)
  }

  async validateUser(id: number) {
    this.logger.log(`validateUser ${id}`)
    return await this.usersService.findOne(id)
  }

  private getAccessToken(userId: number) {
    this.logger.log(`getAccessToken ${userId}`)
    try {
      const payload = {
        id: userId,
      }
      //console.log(payload)
      const access_token = this.jwtService.sign(payload)
      return {
        access_token,
      }
    } catch (error) {
      this.logger.error(error)
      throw new InternalServerErrorException('Error al generar el token')
    }
  }
}
```

## Protegiendo las rutas

Para proteger las rutas haremos uso de nuestros guards: `JwtAuthGuard`, `RolesAuthGuard`. Podemos proteger toda la ruta o solo los métodos que queramos. Por ejemplo, en el siguiente código, protegemos toda la ruta `/categorias` con el guard de autenticación y solo el método `create` con el guard de autorización admin, el resto es para usuarios.

```typescript
@Controller('categorias')
@UseInterceptors(CacheInterceptor) // Aplicar el interceptor aquí de cahce
@UseGuards(JwtAuthGuard, RolesAuthGuard) // Aplicar el guard aquí para autenticados con JWT y Roles (lo aplico a nivel de controlador)
export class CategoriasController {
  private readonly logger = new Logger(CategoriasController.name)

  constructor(private readonly categoriasService: CategoriasService) {}

  @Get()
  @CacheKey('all_categories')
  @CacheTTL(30)
  @Roles('USER')
  async findAll(@Paginate() query: PaginateQuery) {
    this.logger.log('Find all categorias')
    return await this.categoriasService.findAll(query)
  }

  @Get(':id')
  @Roles('USER')
  async findOne(@Param('id', ParseUUIDPipe) id: string) {
    this.logger.log(`Find one categoria by id:${id}`)
    return await this.categoriasService.findOne(id)
  }

  @Post()
  @HttpCode(201)
  @Roles('ADMIN')
  async create(@Body() createCategoriaDto: CreateCategoriaDto) {
    this.logger.log(`Create categoria ${createCategoriaDto}`)
    return await this.categoriasService.create(createCategoriaDto)
  }
}
```

Otro ejemplo, en el siguiente código,la ruta está abierta pero el método `create` solo lo pueden usar los usuarios autenticados con JWT y con rol de admin.

```typescript	
@Controller('productos')
@UseInterceptors(CacheInterceptor) // Aplicar el interceptor aquí de cahce
export class ProductosController {
  private readonly logger: Logger = new Logger(ProductosController.name)

  constructor(private readonly productosService: ProductosService) {}

  @Get()
  @CacheKey('all_products')
  @CacheTTL(30)
  async findAll(@Paginate() query: PaginateQuery) {
    this.logger.log('Find all productos')
    return await this.productosService.findAll(query)
  }

  @Get(':id')
  async findOne(@Param('id') id: number) {
    this.logger.log(`Find one producto by id:${id}`)
    return await this.productosService.findOne(id)
  }

  @Post()
  @HttpCode(201)
  @UseGuards(JwtAuthGuard, RolesAuthGuard) // Aplicar el guard aquí
  @Roles('ADMIN')
  async create(@Body() createProductoDto: CreateProductoDto) {
    this.logger.log(`Create producto ${createProductoDto}`)
    return await this.productosService.create(createProductoDto)
  }
}
```

# SSL y HTTPS
Para usar SSL y HTTPS en NestJS, lo primero que tenemos que hacer es generar un certificado autofirmado. Esta vez vamos a usar OpenSSL para generar el certificado. Para ello, vamos a ejecutar el siguiente comando:

```bash
```bash
openssl genrsa -out keystore.p12 2048
openssl req -new -x509 -key keystore.p12 -out cert.pem -days 365
```

Posteriormente añadiremos el certificado a nuestro proyecto. Para ello, vamos a crear una carpeta llamada `cert` y copiamos los ficheros `keystore.p12` y `cert.pem` en ella.

Desde el fichero .env, vamos a añadir las siguientes variables de entorno:

```bash
SSL_KEY=./cert/keystore.p12
SSL_CERT=./cert/cert.pem
```

Finalmente, vamos a modificar el fichero `main.ts` para que use el certificado. Para ello, vamos a añadir el siguiente código:

```typescript
async function bootstrap() {
  // Leemos la configuración de los certificados SSL
  const httpsOptions = {
    key: readFileSync(path.resolve(process.env.SSL_KEY)),
    cert: readFileSync(path.resolve(process.env.SSL_CERT)),
  }
  const app = await NestFactory.create(AppModule, { httpsOptions })
  // Configuración de la versión de la API
  app.setGlobalPrefix(process.env.API_VERSION || 'v1')
  // Activamos las validaciones body y dtos
  app.useGlobalPipes(new ValidationPipe())
  // Configuración del puerto de escucha
  await app.listen(process.env.API_PORT || 3000)
}

// Inicialización de la aplicación y cuando esté lista se muestra un mensaje en consola
bootstrap().then(() =>
  console.log(
    `🟢 Servidor escuchando en puerto: ${
      process.env.API_PORT || 3000
    } y perfil: ${process.env.NODE_ENV} 🚀`,
  ),
)
```


## Testeando la seguridad
A nivel unitario, nuestro servicios y controladores no se ven modificados por la seguridad. Por lo que no tenemos que hacer nada especial para testearlos.

Pero a nivel de integración con SuperTest, si que tenemos que hacer algunas modificaciones. Son sutil,es pero le vamos a decir que no aplique los guardas en los test ya generados
```typescript
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
      imports: [CacheModule.register()], // importamos el módulo de caché, lo necesita el controlador (interceptores y anotaciones)
      controllers: [CategoriasController],
      providers: [
        CategoriasService,
        { provide: CategoriasService, useValue: mockCategoriasService }, // Mockeamos el servicio
        // Tambien podemos mockear los guardas aquí
        /*
        {
          provide: JwtAuthGuard,
          useValue: { canActivate: () => true },
        },
        {
          provide: RolesAuthGuard,
          useValue: { canActivate: () => true },
        },
        */
      ],
    })
      .overrideGuard(JwtAuthGuard)
      .useValue({ canActivate: () => true }) // Esto permite que todas las solicitudes pasen el JwtAuthGuard
      .overrideGuard(RolesAuthGuard)
      .useValue({ canActivate: () => true }) // Esto permite que todas las solicitudes pasen el RolesAuthGuard
      .compile()

    app = moduleFixture.createNestApplication()
    await app.init()
  })

  afterAll(async () => {
    await app.close()
  })
```

# Práctica de clase: Seguridad
1. Crea todo el modelo de seguridad para que un usuario pueda registrarse y loguearse en el sistema.
2. Los funkos solo pueden ser creados, modificados y eliminados por un usuario administrador. Pueden verse por todo el mundo.
3. Las categorías solo pueden ser creadas, modificadas y eliminadas por un usuario administrador.
4. Los pedidos solo pueden ser creados, modificados y eliminados por un usuario administrador.
5. Un usuario puede ver y modificar su perfil y sus pedidos.

# Proyecto
Puedes consultar esta parte en [el proyecto de ejemplo](https://github.com/joseluisgs/DesarrolloWebEntornosServidor-03-Proyecto-2023-2024/releases/tag/autenticaci%C3%B3n_autorizaci%C3%B3n).


