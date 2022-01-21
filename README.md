# prisma-implicit-explicit-relations

Steps to migrate a prisma schema from implicit to explicit relationshsips. 

This repo holds commits for the individual steps:



## Implicit

This is the starting point where the models `Post` and `Category` have an implicit relationship managed by Prisma. 

```
model Post {
  id         Int        @id @default(autoincrement())
  author     User       @relation(fields: [authorId], references: [id])
  authorId   Int // relation scalar field  (used in the `@relation` attribute above)
  categories Category[]
}

model Category {
  id    Int    @id @default(autoincrement())
  posts Post[]
}
```

The database holds a table `_CategoryToPost` which Prisma uses to manage the relationship 

```bash
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | Category           | table | postgres
 public | Post               | table | postgres
 public | _CategoryToPost    | table | postgres
 public | _prisma_migrations | table | postgres
(4 rows)
```

## Explicit additionally 

Adding fields and a model for an explicit relationship. For clarity the fields used for the implicit relationship were renamed to `categoriesImplicit` and `postsImplicit` respectively. 

```
model Post {
  id         Int        @id @default(autoincrement())
  author     User       @relation(fields: [authorId], references: [id])
  authorId   Int // relation scalar field  (used in the `@relation` attribute above)
  categoriesImplicit Category[]
  categoriesExplicit CategoriesOnPosts[]

}

model Category {
  id    Int    @id @default(autoincrement())
  postsImplicit Post[]
  postsExplicit CategoriesOnPosts[]
}

model CategoriesOnPosts {
  post       Post     @relation(fields: [postId], references: [id])
  postId     Int // relation scalar field (used in the `@relation` attribute above)
  category   Category @relation(fields: [categoryId], references: [id])
  categoryId Int // relation scalar field (used in the `@relation` attribute above)
  assignedAt DateTime @default(now())

  @@id([postId, categoryId])
}
```

Running `npx prisma migrate dev` updates the database and creates a new table `CategoriesOnPosts` for the explicit relationship.


```bash
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | CategoriesOnPosts  | table | postgres
 public | Category           | table | postgres
 public | Post               | table | postgres
 public | _CategoryToPost    | table | postgres
 public | _prisma_migrations | table | postgres
(5 rows)
```

## Remove implicit

Remove the implicit fields from the Prisma schema:

```
model Post {
  id         Int        @id @default(autoincrement())
  author     User       @relation(fields: [authorId], references: [id])
  authorId   Int // relation scalar field  (used in the `@relation` attribute above)
  categoriesExplicit CategoriesOnPosts[]

}

model Category {
  id    Int    @id @default(autoincrement())
  postsExplicit CategoriesOnPosts[]
}

model CategoriesOnPosts {
  post       Post     @relation(fields: [postId], references: [id])
  postId     Int // relation scalar field (used in the `@relation` attribute above)
  category   Category @relation(fields: [categoryId], references: [id])
  categoryId Int // relation scalar field (used in the `@relation` attribute above)
  assignedAt DateTime @default(now())

  @@id([postId, categoryId])
}

```

Running `npx prisma migrate dev` updates the database and removes the table `_CategoryToPost` as its not needed anymore.


```bash
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | CategoriesOnPosts  | table | postgres
 public | Category           | table | postgres
 public | Post               | table | postgres
 public | _prisma_migrations | table | postgres
(4 rows)
```


## Rename fields

Now we can rename the fields for the explicit relationship as if nothing had changed. This change does not impact the database as the fields are only represented in the Prisma schema.

```
model Post {
  id         Int        @id @default(autoincrement())
  author     User       @relation(fields: [authorId], references: [id])
  authorId   Int // relation scalar field  (used in the `@relation` attribute above)
  categories CategoriesOnPosts[]

}

model Category {
  id    Int    @id @default(autoincrement())
  posts CategoriesOnPosts[]
}

model CategoriesOnPosts {
  post       Post     @relation(fields: [postId], references: [id])
  postId     Int // relation scalar field (used in the `@relation` attribute above)
  category   Category @relation(fields: [categoryId], references: [id])
  categoryId Int // relation scalar field (used in the `@relation` attribute above)
  assignedAt DateTime @default(now())

  @@id([postId, categoryId])
}
```


