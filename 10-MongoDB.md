- [Bases de Datos NoSQL: MongoDB](#bases-de-datos-nosql-mongodb)
  - [Instalación](#instalación)
  - [Configuración](#configuración)
  - [Creación de Esquemas y Modelos de Documentos](#creación-de-esquemas-y-modelos-de-documentos)
  - [Servicios](#servicios)
  - [Testing](#testing)
- [Práctica de clase: MongoDB](#práctica-de-clase-mongodb)
- [Proyecto](#proyecto)

![](./images/10-banner.png)

# Bases de Datos NoSQL: MongoDB
MongoDB es una base de datos NoSQL orientada a documentos, de código abierto y de uso general. Esto nos permite trabajar consiguiendo las ventajas de las bases de datos NoSQL, como la adaptación al cambio en la colecciones, la escalabilidad y la flexibilidad, pero manteniendo la potencia y la funcionalidad de las bases de datos tradicionales.

## Instalación
En este apartado vamos a ver como usar [MongoDB](https://mongoosejs.com/) en nuestro proyecto Nestjs y hacer la [paginación](https://github.com/aravindnc/mongoose-paginate-v2?tab=readme-ov-file). Para ello vamos a instalar las siguientes dependencias:

```bash
$ npm install --save @nestjs/mongoose mongoose
$ npm install --save mongoose-paginate-v2
```

## Configuración
Nuestro siguiente paso es configurar la conexión a la base de datos. De igual manera que hemos hecho en otras ocasiones, podemos hacerlo en el fichero `app.module.ts`:

```typescript
@Module({
  imports: [
    UsersModule,
    ProductsModule,
    // Configuración para la conexión a la base de datos a MongoDB
    MongooseModule.forRoot(
      'mongodb://admin:adminPassword123@localhost:27017/NEST_DB', // Dirección de la base de datos
    ),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

## Creación de Esquemas y Modelos de Documentos
Con MongoDB, los datos se almacenan en documentos JSON. Estos documentos se almacenan en colecciones, que a su vez se almacenan en bases de datos. Mientras con TypeORM teníamos documentos, en MongoDB tenemos documentos JSON. 

Lo primero es crear el esquema con el decorador `@Schema()` y los campos que tendrá nuestro documento. Para ello, vamos a crear un esquema para la colección `countrylanguage` que tiene los siguientes campos:

```ts
@Schema()
export class CountryLanguage {
  @Prop({ type: String, required: true })
  countrycode: string

  @Prop({ type: String, required: true })
  language: string

  @Prop({ type: Boolean, required: true })
  isofficial: boolean

  @Prop({ type: Number, required: true })
  percentage: number
}
```

Luego hacemos uso de SchemaFactory para crear el esquema con el plugin de paginación, de esta manera podemos paginar los resultados de las consultas (lo extendemos):

```ts
export const CountryLanguageSchema =
  SchemaFactory.createForClass(CountryLanguage)
CountryLanguageSchema.plugin(mongoosePaginate)
```
Finalmente, creamos el tipo del documento que nos servirá para mapear los resultados paginados usando la intersección de tipos (&) que hace que se combinen multiples tipos en uno.
  
```ts
export type CountryLanguageDocument = CountryLanguage & Document
```

Finalmente en el módulo de donde vamos a usar importamos el recurso para usarlos en los servicios:

```ts
@Module({
  imports: [
    MongooseModule.forFeatureAsync([
      {
        name: CountryLanguage.name,
        useFactory: () => {
          const schema = SchemaFactory.createForClass(CountryLanguage)
          schema.plugin(mongoosePaginate)
          return schema
        },
      },
    ]),
  ],
  controllers: [CountriesLanguagesController],
  providers: [CountriesLanguagesService],
})
export class CountriesLanguagesModule {}
```

## Servicios
Podemos crear servicios para trabajar con los documentos de la base de datos. Para ello, vamos a crear un servicio para la colección `countrylanguage` que tiene los siguientes métodos y algunos de ellos paginados usando las distintas opciones que nos ofrece el [plugin de paginación de Mongoose](https://github.com/aravindnc/mongoose-paginate-v2?tab=readme-ov-file):

```ts
@Injectable()
export class CountriesLanguagesService {
  private logger = new Logger(CountriesLanguagesService.name)

  constructor(
    @InjectModel(CountryLanguage.name)
    private countryLanguageModel: PaginateModel<CountryLanguageDocument>,
  ) {}

  async findAll(): Promise<CountryLanguage[]> {
    return await this.countryLanguageModel.find().exec()
  }

  async findAllPaginated(
    page: number,
    pageSize: number,
    filter: CountryLanguageFilter,
    order: CountryLanguageOrder,
    search: string,
  ) {
    this.logger.log(
      `page: ${page}, pageSize: ${pageSize}, filter: ${filter}, order: ${order}, search: ${search}`,
    )
    // Aquí iría la query de búsqueda y filtrado
    const query = {
      [filter]: {
        $regex: `.*${search}.*`, // para que busque en cualquier parte del campo
        $options: 'i', // para que no distinga entre mayúsculas y minúsculas
      },
    }
    // Aquí iría la query de ordenación y paginación
    const options = {
      page,
      limit: pageSize,
      sort: { [filter]: order },
      collection: 'es_ES', // para que use la configuración de idioma de España
    }
    // lanzamos la operación de búsqueda y paginación
    // si no hay filtro, query será un objeto vacío, y no puede ser Percentage
    return await this.countryLanguageModel.paginate(
      filter !== 'Percentage' ? query : {},
      options,
    )
  }
}
```

## Testing
De la misma manera que hemos hecho con TypeORM, podemos hacer test de los servicios de MongoDB. Mockeando los elementos y métodos necesarios con spyOn y mockReturnValue. Por ejemplo, para el método `findAllPaginated`:

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { getModelToken } from '@nestjs/mongoose';
import { CountriesLanguagesService } from './countries-languages.service';
import { CountryLanguage, CountryLanguageDocument } from './country-language.schema';
import { PaginateModel } from 'mongoose';

describe('CountriesLanguagesService', () => {
  let service: CountriesLanguagesService;
  let model: PaginateModel<CountryLanguageDocument>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        CountriesLanguagesService,
        {
          provide: getModelToken(CountryLanguage.name),
          useValue: {
            find: jest.fn(),
            paginate: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<CountriesLanguagesService>(CountriesLanguagesService);
    model = module.get<PaginateModel<CountryLanguageDocument>>(getModelToken(CountryLanguage.name));
  });

  describe('findAll', () => {
    it('should return an array of CountryLanguage', async () => {
      const mockCountryLanguages: CountryLanguage[] = [
        { name: 'English', population: 100 },
        { name: 'Spanish', population: 200 },
      ];

      jest.spyOn(model, 'find').mockReturnValueOnce({
        exec: jest.fn().mockResolvedValueOnce(mockCountryLanguages),
      } as any);

      const result = await service.findAll();

      expect(result).toEqual(mockCountryLanguages);
      expect(model.find).toHaveBeenCalledTimes(1);
    });
  });

  describe('findAllPaginated', () => {
    it('should return paginated results', async () => {
      const page = 1;
      const pageSize = 10;
      const filter = 'name';
      const order = 'asc';
      const search = 'English';

      const mockPaginatedResult = {
        docs: [{ name: 'English', population: 100 }],
        totalDocs: 1,
        limit: pageSize,
        page: page,
        totalPages: 1,
        pagingCounter: 1,
        hasPrevPage: false,
        hasNextPage: false,
        prevPage: null,
        nextPage: null,
      };

      jest.spyOn(model, 'paginate').mockResolvedValueOnce(mockPaginatedResult as any);

      const result = await service.findAllPaginated(page, pageSize, filter, order, search);

      expect(result).toEqual(mockPaginatedResult);
      expect(model.paginate).toHaveBeenCalledTimes(1);
      expect(model.paginate).toHaveBeenCalledWith(
        { name: { $regex: `.*${search}.*`, $options: 'i' } },
        { page, limit: pageSize, sort: { name: order }, collection: 'es_ES' },
      );
    });
  });
});
```

# Práctica de clase: MongoDB
1. Crea un Pedido que estará compuesto de un Cliente con su dirección y de distintas Líneas de Pedido. El pedido debe calcular el total de items y el precio total. Se debe almacenar en MongoDB.
2. Crea un repositorios y servicio que permita crear, leer, actualizar y eliminar pedidos.
Se debe tener en cuenta que no se puede añadir un Funko si no hay stock suficiente y que al devolverlo, se debe ajustar el stock.
3. Recuerda añadir consultas con paginación y búsqueda personalizadas.
4. Crea los test necesarios de repositorio (si los hay), servicio y controlador.

# Proyecto
Puedes consultar esta parte en [el proyecto de ejemplo](https://github.com/joseluisgs/DesarrolloWebEntornosServidor-03-Proyecto-2023-2024/releases/tag/pedidos).

