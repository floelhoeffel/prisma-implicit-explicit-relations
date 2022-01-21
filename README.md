# prisma-implicit-explicit-relations

## Implicit

```
model User {
  id      Int      @id @default(autoincrement())
  posts   Post[]
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  user   User @relation(fields: [userId], references: [id])
  userId Int // relation scalar field (used in the `@relation` attribute above)
}

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

```bash
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | Category           | table | postgres
 public | Post               | table | postgres
 public | Profile            | table | postgres
 public | User               | table | postgres
 public | _CategoryToPost    | table | postgres
 public | _prisma_migrations | table | postgres
(6 rows)
```

## Explicit additionally 

```
model User {
  id      Int      @id @default(autoincrement())
  posts   Post[]
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  user   User @relation(fields: [userId], references: [id])
  userId Int // relation scalar field (used in the `@relation` attribute above)
}

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



```bash
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | CategoriesOnPosts  | table | postgres
 public | Category           | table | postgres
 public | Post               | table | postgres
 public | Profile            | table | postgres
 public | User               | table | postgres
 public | _CategoryToPost    | table | postgres
 public | _prisma_migrations | table | postgres
(7 rows)
```