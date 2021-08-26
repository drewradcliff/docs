---
id: prisma
title: Prisma Adapter
---

# Prisma

To use this Adapter, you need to install Prisma Client, Prisma CLI, and the separate `@next-auth/prisma-adapter` package:

```
npm install @prisma/client @next-auth/prisma-adapter
npm install prisma --save-dev
```

Configure your NextAuth.js to use the Prisma Adapter:

```javascript title="pages/api/auth/[...nextauth].js"
import NextAuth from "next-auth"
import Providers from "next-auth/providers"
import { PrismaAdapter } from "@next-auth/prisma-adapter"
import { PrismaClient } from "@prisma/client"

const prisma = new PrismaClient()

export default NextAuth({
  providers: [
    Providers.Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
  ],
  adapter: PrismaAdapter(prisma),
})
```

:::tip
While Prisma includes an experimental feature in the migration command that is able to generate SQL from a schema, creating tables and columns using the provided SQL is currently recommended instead as SQL schemas automatically generated by Prisma may differ from the recommended schemas.
:::
Schema for the Prisma Adapter (`@next-auth/prisma-adapter`)

## Setup

You need to use at least Prisma 2.26.0. Create a schema file in `prisma/schema.prisma` similar to this one:

```json title="schema.prisma"
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["referentialActions"]
}

model Account {
  id                 String  @id @default(cuid())
  userId             String
  type               String
  provider           String
  providerAccountId  String
  refresh_token      String?
  access_token       String?
  expires_at         Int?
  token_type         String?
  scope              String?
  id_token           String?
  session_state      String?
  oauth_token_secret String?
  oauth_token        String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  accounts      Account[]
  sessions      Session[]
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```

### Generate Client

Once you have saved your schema, use the Prisma CLI to generate the Prisma Client:

```
npx prisma generate
```

To configure you database to use the new schema (i.e. create tables and columns) use the `prisma migrate` command:

```
npx prisma migrate dev
```

To generate a schema in this way with the above example code, you will need to specify your database connection string in the environment variable `DATABASE_URL`. You can do this by setting it in a `.env` file at the root of your project.

As this feature is experimental in Prisma, it is behind a feature flag. You should check your database schema manually after using this option. See the [Prisma documentation](https://www.prisma.io/docs/) for information on how to use `prisma migrate`.