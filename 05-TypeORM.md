- [Repositorios SQL con TypeORM](#repositorios-sql-con-typeorm)
  - [Instalación y configuración](#instalación-y-configuración)
  - [Entidades](#entidades)
  - [Relaciones](#relaciones)
  - [Repositorios](#repositorios)
- [Práctica de clase: TypeORM y SQL](#práctica-de-clase-typeorm-y-sql)
- [Proyecto](#proyecto)

![](images/05-banner.png)

# Repositorios SQL con TypeORM

## Instalación y configuración
Para trabajar con Bases de Datos nos vamos a ayudar de Docker y Docker Compose y sobre todo de TypeORM, que es on ORM para JS/TypeScript y compatible totalmente con Nest.js y nos [implementa el patrón Repository](https://docs.nestjs.com/recipes/sql-typeorm).

Lo primero es instalar su módulo y las dependencias a TypeORM y a cada uno de los SGDB que usemos, por ejemplo, para PostgreSQL:

```bash
npm install --save @nestjs/typeorm typeorm pg
```

Luego configuramos la conexión en nuestro app.module.ts

```ts
@Module({
  imports: [
    UsersModule,
    ProductsModule,
    // Configuración de la conexión a la base de datos a PostgreSQL
    TypeOrmModule.forRoot({
      type: 'postgres', // Tipo de base de datos
      host: 'localhost', // Dirección del servidor
      port: 5432, // Puerto del servidor
      username: 'admin', // Nombre de usuario
      password: 'adminPassword123', // Contraseña de usuario
      database: 'NEST_DB', // Nombre de la base de datos
      entities: [__dirname + '/**/*.entity{.ts,.js}'], // Entidades de la base de datos (buscar archivos con extensión .entity.ts o .entity.js)
      synchronize: true, // Sincronizar la base de datos
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

## Entidades
Debemos crear las entidades de datos, que son las clases que representan las tablas de la base de datos en base a anotaciones de TypeORM cons sus opciones si no queremos que tomen las opciones por defecto. Por ejemplo, para un usuario:

```ts
@Entity('users') // Nombre de la tabla
export class User {
  @PrimaryGeneratedColumn() // Columna de clave primaria autoincrementable
  id: number

  @Column('varchar', { length: 255, nullable: false, name: 'first_name' })
  firstname: string

  @Column('varchar', { length: 255, nullable: false, name: 'last_name' })
  lastname: string

  @Column('varchar', { length: 255, nullable: true, default: 'no address' })
  address: string

  @Column('varchar', {
    length: 150,
    nullable: false,
    name: 'single_status',
    default: false,
  })
  single: boolean
}
```

## Relaciones
Podemos usar las [anotaciones de TypeORM para definir las relaciones entre tablas](https://typeorm.io/relations). Estas aceptan un callback de configuración de cada realacion Entre ellas tenemos las siguientes:
- OneToOne
- OneToMany
- ManyToMany

De nuevo es interesante tener en cuenta la bidireccionalidad de las mimas.

``En TypeORM, para establecer relaciones entre entidades utilizando el framework Nest, se pueden utilizar decoradores como `@ManyToOne`, `@OneToMany`, `@ManyToMany`, entre otros. En el caso específico que mencionaste, donde un producto tiene una categoría y una categoría tiene muchos productos, puedes definir las relaciones de la siguiente manera:

1. Primero, vamos a crear la entidad "Categoría":

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { Producto } from './producto.entity';

@Entity()
export class Categoria {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nombre: string;

  @OneToMany(() => Producto, (producto) => producto.categoria)
  productos: Producto[];
}
```

2. Luego, crearemos la entidad "Producto":

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, JoinColumn } from 'typeorm';
import { Categoria } from './categoria.entity';

@Entity()
export class Producto {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nombre: string;

  @ManyToOne(() => Categoria, (categoria) => categoria.productos)
  @JoinColumn({ name: 'categoria_id' }) // Especifica el nombre de la columna
  categoria: Categoria;
}
```

En este ejemplo, la entidad "Producto" tiene una relación `@ManyToOne` con la entidad "Categoria". Esto significa que un producto pertenece a una sola categoría.

Por otro lado, la entidad "Categoria" tiene una relación `@OneToMany` con la entidad "Producto". Esto indica que una categoría puede tener muchos productos relacionados.

Tener la relación bidireccional (es decir, establecer tanto `@ManyToOne` como `@OneToMany` en ambas entidades) es opcional. Dependiendo de tus necesidades, puedes decidir si necesitas o no la bidireccionalidad en este caso. Sin embargo, la bidireccionalidad puede facilitar el acceso y la navegación entre las entidades relacionadas.

En este caso, se utiliza `@JoinColumn` con la opción name para establecer el nombre de la columna en la tabla de Producto que representa la relación con la categoría. En este ejemplo, se fija el nombre de la columna como 'categoria_id'.

`@JoinColumn` se utiliza para indicar a TypeORM cómo deben relacionarse las tablas en la base de datos mediante las claves foráneas. Especificar @JoinColumn junto con @ManyToOne proporciona a TypeORM la información necesaria para crear correctamente la relación.

Para implementar la cascada en las relaciones entre entidades en TypeORM, puedes utilizar el decorador `@OneToMany` o `@ManyToOne` en conjunto con la opción `cascade` para especificar las acciones que deseas realizar en cascada.

Por ejemplo, si deseas que al eliminar una categoría se eliminen automáticamente todos los productos asociados, puedes configurar la cascada de la siguiente manera:

En la entidad `Categoria`:
```typescript
@OneToMany(() => Producto, (producto) => producto.categoria, { cascade: true })
productos: Producto[];
```

En la entidad `Producto`:
```typescript
@ManyToOne(() => Categoria, (categoria) => categoria.productos, { onDelete: 'CASCADE' })
@JoinColumn({ name: 'categoria_id' })
categoria: Categoria;
```
En este ejemplo, se utiliza la opción `cascade` en `@OneToMany` para especificar que las acciones de eliminación en cascada se deben aplicar a la relación. Al establecer `{ cascade: true }`, cuando se elimina una categoría, se eliminarán automáticamente todos los productos relacionados.

Adicionalmente, también se utiliza `{ onDelete: 'CASCADE' }` en `@ManyToOne` para indicar que cuando se elimine una categoría, se deben eliminar automáticamente los productos asociados a dicha categoría.

Recuerda que al utilizar la cascada, debes tener cuidado con sus implicaciones. Asegúrate de comprender plenamente el impacto de estas acciones y cómo afectarán a tu sistema antes de aplicarlas.

## Repositorios
Para usar el repositorio de la entidad que quieras debes importarlo a tu módulo:
  
```ts
  import { TypeOrmModule } from '@nestjs/typeorm';

  @Module({
    imports: [
      TypeOrmModule.forFeature([User]),
    ],
  })
  export class UsersModule {}
```

Luego en nuestro servicio importamos el repositorio de la entidad y lo usamos en los métodos del servicio. Importante los métodos de repositorio devuelven una promesa, por lo que debemos usar async/await o then/catch tanto en servicio como en el controlador.

```js
@Injectable()
export class UsersService {
  // Nos creamos el repositorio de usuarios, que es el que se encarga de la lógica de negocio
  constructor(
    @InjectRepository(User) private readonly userRepository: Repository<User>,
  ) {}


  async findAll() {
    return this.userRepository.find()
  }

  async findOne(id: number) {
    return this.userRepository.findOneBy({ id })
  }

  async remove(id: number) {
    return this.userRepository.delete({ id })
  }
}
```

# Práctica de clase: TypeORM y SQL
1. Crea la entidad Categoría, teniendo en cuenta que un Funko tiene una sola categoria que puede ser: SERIE, DISNEY, SUPERHEROES, PELICULA, OTROS. Una categoria tiene una fecha de creación, de actualización y puede estar activa o no.
2. Crea los endpoints completos para gestionar Funkos y Categorías.

# Proyecto
Puedes consultar esta parte en [el proyecto de ejemplo](https://github.com/joseluisgs/DesarrolloWebEntornosServidor-03-Proyecto-2023-2024/releases/tag/productos_categorias).
