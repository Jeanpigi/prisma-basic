# Prisma

## What is data modeling?

The term data modeling refers to the process of defining the shape and structure of the objects in an application, these objects are often called "application models". In relational databases (like PostgreSQL), they are stored in tables . When using document databases (like MongoDB), they are stored in collections.

Depending on the domain of your application, the models will be different. For example, if you're writing a blogging application, you might have models such as blog, author, article. When writing a carsharing app, you probably have models like driver, car, route. Application models enable you to represent these different entities in your code by creating respective data structures.

When modeling data, you typically ask questions like:

1) What are the main entities/concepts in my application?
2) How do they relate to each other?
3) What are their main characteristics/properties?
4) How can they be represented with my technology stack?

## Data modeling without Prisma

Data modeling typically needs to happen on (at least) two levels:

    On the database level
    On the application level (i.e., in your programming language)

The way how the application models are represented on both levels might differ due to a few reasons:

    Databases and programming languages use different data types
    Relations are represented differently in a database than in a programming language
    Databases typically have more powerful data modeling capabilities, like indexes, cascading deletes, or a variety of additional constraints (e.g. unique, not null, ...)
    Databases and programming languages have different technical constraints

# Data modeling with Prisma

Depending on which parts of Prisma you want to use in your application, the data modeling flow looks slightly different. The following two sections explain the workflows for using only Prisma Client and using Prisma Client and Prisma Migrate.

No matter which approach though, with Prisma you never create application models in your programming language by manually defining classes, interfaces, or structs. Instead, the application models are defined in your Prisma schema:

    Only Prisma Client: Application models in the Prisma schema are generated based on the introspection of your database schema. Data modeling happens primarily on the database-level.
    Prisma Client and Prisma Migrate: Data modeling happens in the Prisma schema by manually adding application models to it. Prisma Migrate maps these application models to tables in the underlying database (currently only supported for relational databases).

As an example, the User model from the previous example would be represented as follows in the Prisma schema:

```
    model User {
        user_id Int     @id @default(autoincrement())
        name    String?
        email   String  @unique
        isAdmin Boolean @default(false)
    }
```
Once the application models are in your Prisma schema (whether they were added through introspection or manually by you), the next step typically is to generate Prisma Client which provides a programmatic and type-safe API to read and write data in the shape of your application models.

Prisma Client uses TypeScript type aliases to represent your application models in your code. For example, the User model would be represented as follows in the generated Prisma Client library:

```
    export declare type User = {
        id: number
        name: string | null
        email: string
        isAdmin: boolean
    }
```

# Prisma schema

The Prisma schema file (short: schema file, Prisma schema or schema) is the main configuration file for your Prisma setup. It is typically called schema.prisma and consists of the following parts:

1) Data sources: Specify the details of the data sources Prisma should connect to (e.g. a PostgreSQL database).
2) Generators: Specifies what clients should be generated based on the data model (e.g. Prisma Client).
3) Data model definition: Specifies your application models (the shape of the data per data source) and their relations.

Whenever a prisma command is invoked, the CLI typically reads some information from the schema file, e.g.:

    prisma generate: Reads all above mentioned information from the Prisma schema to generate the correct data source client code (e.g. Prisma Client).
    prisma migrate dev: Reads the data sources and data model definition to create a new migration.

You can also use environment variables inside the schema file to provide configuration options when a CLI command is invoked.

# Example

The following is an example of a Prisma schema file that specifies:

    A data source (PostgreSQL or MongoDB)
    A generator (Prisma Client)
    A data model definition with two models (with one relation) and one enum
    Several native data type attributes (@db.VarChar(255), @db.ObjectId)

```
    datasource db {
        url      = env("DATABASE_URL")
        provider = "postgresql"
    }

    generator client {
        provider = "prisma-client-js"
    }

    model User {
        id        Int      @id @default(autoincrement())
        createdAt DateTime @default(now())
        email     String   @unique
        name      String?
        role      Role     @default(USER)
        posts     Post[]
    }

    model Post {
        id        Int      @id @default(autoincrement())
        createdAt DateTime @default(now())
        updatedAt DateTime @updatedAt
        published Boolean  @default(false)
        title     String   @db.VarChar(255)
        author    User?    @relation(fields: [authorId], references: [id])
        authorId  Int?
    }

    enum Role {
        USER
        ADMIN
    }
```

## Accessing environment variables from the schema

You can use environment variables to provide configuration options when a CLI command is invoked, or a Prisma Client query is run.

Using environment variables in the schema allows you to keep secrets out of the schema file which in turn improves the portability of the schema by allowing you to use it in different environments

Environment variables can be accessed using the env() function:

```
    datasource db {
        provider = "postgresql"
        url      = env("DATABASE_URL")
    }
```


## .env
DATABASE_URL=postgresql://test:test@localhost:5432/test
DATABASE_URL_WITH_SCHEMA=${DATABASE_URL}?schema=public

## How to use Prisma

By terminal, just write the following command:

```
    npx prisma studio
```
## Prisma Studio

Prisma Studio is a visual editor for the data in your database. Note that Prisma Studio is not open source but you can still create issues in the prisma/studio repo.

## Prisma Migrate

Prisma Migrate is an imperative database schema migration tool that enables you to:

    Keep your database schema in sync with your Prisma schema as it evolves and
    Maintain existing data in your database

Prisma Migrate generates a history of .sql migration files, and plays a role in both development and deployment.

To get started with Prisma Migrate in a development environment:

```
    datasource db {
        provider = "postgresql"
        url      = env("DATABASE_URL")
    }


    model User {
        id    Int    @id @default(autoincrement())
        name  String
        posts Post[]
    }


    model Post {
        id        Int     @id @default(autoincrement())
        title     String
        published Boolean @default(true)
        authorId  Int
        author    User    @relation(fields: [authorId], references: [id])
    }
```

Create the first migration:

```
 prisma migrate dev --name init or npx prisma migrate dev --name init
```

Learn more about this file in the docs: https://pris.ly/d/prisma-schema