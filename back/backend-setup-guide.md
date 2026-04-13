# Guide d'initialisation — Backend NestJS
## Projet Lidl Drive

> Mode d'emploi complet pour démarrer le projet backend from scratch.
> Stack : NestJS + TypeScript + PostgreSQL + TypeORM + Swagger

---

## Prérequis

Avant de commencer, vérifie que tu as installé :

- **Node.js** (version 18 ou supérieure) → [nodejs.org](https://nodejs.org)
- **npm** (installé automatiquement avec Node)
- **Docker** (pour PostgreSQL en local) → [docker.com](https://docker.com)

Pour vérifier :

```bash
node -v    # doit afficher v18.x.x ou supérieur
npm -v     # doit afficher une version
docker -v  # doit afficher une version
```

---

## Étape 1 — Installer le CLI NestJS

Le CLI de NestJS permet de générer automatiquement la structure du projet et les fichiers.

```bash
npm install -g @nestjs/cli
```

Vérifie l'installation :

```bash
nest --version
```

---

## Étape 2 — Créer le projet

```bash
nest new lidl-drive-backend
```

Le CLI va te demander quel gestionnaire de paquets utiliser. **Choisis npm**.

Une fois créé, entre dans le dossier :

```bash
cd lidl-drive-backend
```

---

## Étape 3 — Structure générée

Tu dois voir ceci :

```
lidl-drive-backend/
├── src/
│   ├── app.controller.ts     ← controller par défaut (tu peux le supprimer plus tard)
│   ├── app.module.ts         ← module racine de l'application
│   ├── app.service.ts        ← service par défaut (tu peux le supprimer plus tard)
│   └── main.ts               ← point d'entrée de l'application
├── test/                     ← tests e2e
├── .eslintrc.js
├── .prettierrc
├── nest-cli.json
├── package.json
├── tsconfig.json
└── tsconfig.build.json
```

---

## Étape 4 — Installer les dépendances nécessaires

### Base de données (TypeORM + PostgreSQL)

```bash
npm install @nestjs/typeorm typeorm pg
```

- `@nestjs/typeorm` → intégration NestJS
- `typeorm` → l'ORM (équivalent JPA/Hibernate)
- `pg` → le driver PostgreSQL

### Variables d'environnement

```bash
npm install @nestjs/config
```

Permet de lire les variables depuis un fichier `.env`.

### Validation des données

```bash
npm install class-validator class-transformer
```

Permet de valider automatiquement les données reçues dans les requêtes (équivalent `@Valid` en Spring).

### Swagger

```bash
npm install @nestjs/swagger
```

### Sécurité (JWT pour l'auth — à faire plus tard)

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install -D @types/passport-jwt
```

> Tu n'as pas besoin de configurer l'auth maintenant. Installe juste les paquets.

---

## Étape 5 — Créer le fichier .env

À la racine du projet, crée un fichier `.env` :

```env
# Application
PORT=3000
NODE_ENV=development

# Base de données
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=lidl_user
DB_PASSWORD=lidl_password
DB_NAME=lidl_drive

# JWT (pour plus tard)
JWT_SECRET=change_this_secret_before_production
JWT_EXPIRES_IN=1h
```

> **Important** : ajoute `.env` dans ton `.gitignore` pour ne jamais committer ce fichier.

```bash
echo ".env" >> .gitignore
```

---

## Étape 6 — Lancer PostgreSQL avec Docker

Crée un fichier `docker-compose.yml` à la racine du projet :

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: lidl_drive_db
    environment:
      POSTGRES_USER: lidl_user
      POSTGRES_PASSWORD: lidl_password
      POSTGRES_DB: lidl_drive
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Lance la base de données :

```bash
docker compose up -d
```

Vérifie que le conteneur tourne :

```bash
docker ps
```

Tu dois voir `lidl_drive_db` dans la liste avec le statut `Up`.

---

## Étape 7 — Configurer l'application

### Modifier `src/app.module.ts`

Remplace le contenu par :

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    // Charge les variables d'environnement depuis .env
    ConfigModule.forRoot({
      isGlobal: true, // disponible dans toute l'application sans réimporter
    }),

    // Connexion à PostgreSQL
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get<number>('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: true, // ⚠️ pratique en dev, à désactiver en production
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

> **Note sur `synchronize: true`** : TypeORM va créer/modifier automatiquement les tables en base selon tes entités. C'est pratique pour le développement mais dangereux en production (tu pourrais perdre des données). On le désactive plus tard.

### Modifier `src/main.ts`

Remplace le contenu par :

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { ConfigService } from '@nestjs/config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const configService = app.get(ConfigService);
  const port = configService.get<number>('PORT') || 3000;

  // Préfixe global pour toutes les routes → /api/...
  app.setGlobalPrefix('api');

  // Active la validation automatique des DTOs sur toutes les routes
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,       // supprime les champs non déclarés dans le DTO
      forbidNonWhitelisted: true, // renvoie une erreur si champ inconnu
      transform: true,       // convertit automatiquement les types (string → number etc.)
    }),
  );

  // Configuration Swagger
  const config = new DocumentBuilder()
    .setTitle('Lidl Drive API')
    .setDescription('API du projet Drive Click & Collect')
    .setVersion('1.0')
    .addBearerAuth() // prépare l'auth JWT pour plus tard
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('docs', app, document);

  // CORS (permet au front de communiquer avec l'API)
  app.enableCors();

  await app.listen(port);
  console.log(`Application démarrée sur : http://localhost:${port}`);
  console.log(`Swagger disponible sur   : http://localhost:${port}/docs`);
}
bootstrap();
```

---

## Étape 8 — Démarrer et vérifier

Lance l'application en mode développement :

```bash
npm run start:dev
```

Tu dois voir dans le terminal :

```
Application démarrée sur : http://localhost:3000
Swagger disponible sur   : http://localhost:3000/docs
```

Ouvre le navigateur sur `http://localhost:3000/docs` → tu dois voir l'interface Swagger.

---

## Étape 9 — Générer ton premier module métier

Exemple avec les produits. Le CLI génère automatiquement tous les fichiers :

```bash
nest generate module products
nest generate controller products
nest generate service products
```

Ou en raccourci :

```bash
nest g module products
nest g controller products
nest g service products
```

Tu obtiens :

```
src/
└── products/
    ├── products.module.ts
    ├── products.controller.ts
    └── products.service.ts
```

> NestJS ajoute automatiquement le module `ProductsModule` dans `app.module.ts`. Tu n'as pas à le faire manuellement.

---

## Étape 10 — Créer une première entité

Crée le fichier `src/products/product.entity.ts` :

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

@Entity('products')
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @Column({ nullable: true })
  description: string;

  @Column({ default: true })
  isAvailable: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

Déclare l'entité dans `products.module.ts` :

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsController } from './products.controller';
import { ProductsService } from './products.service';
import { Product } from './product.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Product])], // enregistre l'entité
  controllers: [ProductsController],
  providers: [ProductsService],
})
export class ProductsModule {}
```

---

## Étape 11 — Premier endpoint complet

### Le DTO (`src/products/dto/create-product.dto.ts`)

```typescript
import { IsString, IsNumber, IsOptional, Min } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateProductDto {
  @ApiProperty({ example: 'Pommes Gala', description: 'Nom du produit' })
  @IsString()
  name: string;

  @ApiProperty({ example: 1.99, description: 'Prix en euros' })
  @IsNumber()
  @Min(0)
  price: number;

  @ApiProperty({ example: 'Pommes croquantes', required: false })
  @IsOptional()
  @IsString()
  description?: string;
}
```

### Le service (`src/products/products.service.ts`)

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from './product.entity';
import { CreateProductDto } from './dto/create-product.dto';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private readonly productRepository: Repository<Product>,
  ) {}

  findAll(): Promise<Product[]> {
    return this.productRepository.find();
  }

  findOne(id: number): Promise<Product | null> {
    return this.productRepository.findOneBy({ id });
  }

  create(dto: CreateProductDto): Promise<Product> {
    const product = this.productRepository.create(dto);
    return this.productRepository.save(product);
  }
}
```

### Le controller (`src/products/products.controller.ts`)

```typescript
import { Controller, Get, Post, Body, Param, ParseIntPipe } from '@nestjs/common';
import { ApiTags, ApiOperation } from '@nestjs/swagger';
import { ProductsService } from './products.service';
import { CreateProductDto } from './dto/create-product.dto';

@ApiTags('products')
@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  @ApiOperation({ summary: 'Récupérer tous les produits' })
  findAll() {
    return this.productsService.findAll();
  }

  @Get(':id')
  @ApiOperation({ summary: 'Récupérer un produit par son ID' })
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.productsService.findOne(id);
  }

  @Post()
  @ApiOperation({ summary: 'Créer un nouveau produit' })
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }
}
```

Relance le serveur et ouvre Swagger → tu dois voir les routes `/api/products` documentées et testables.

---

## Récapitulatif des commandes utiles

```bash
# Démarrer en mode dev (hot reload)
npm run start:dev

# Lancer la base de données
docker compose up -d

# Stopper la base de données
docker compose down

# Générer un module complet
nest g module <nom>
nest g controller <nom>
nest g service <nom>

# Voir les logs Docker
docker logs lidl_drive_db
```

---

## Ordre recommandé pour la suite

1. Créer les modules et entités principaux (Store, Product, User, Order...)
2. Configurer l'authentification JWT
3. Ajouter la gestion des rôles (admin / client)
4. Implémenter les règles métier (stock, créneaux, commandes)
5. Dockeriser le backend complet
6. Brancher avec le front

---

## Structure cible du projet (à terme)

```
src/
├── auth/
├── users/
├── stores/
├── products/
├── inventory/
├── cart/
├── orders/
├── slots/
├── notifications/
└── common/           ← guards, interceptors, filtres d'erreurs partagés
```
