# Migration `20200821135747`

This migration has been generated by Daniel Norman at 8/21/2020, 3:57:47 PM.
You can check out the [state of the schema](./schema.prisma) after the migration.

## Database Steps

```sql
CREATE TYPE "UserRole" AS ENUM ('STUDENT', 'TEACHER');

CREATE TYPE "TokenType" AS ENUM ('EMAIL', 'API');

CREATE TABLE "public"."User" (
"id" SERIAL,
"email" text  NOT NULL ,
"firstName" text   ,
"lastName" text   ,
"social" jsonb   ,
PRIMARY KEY ("id"))

CREATE TABLE "public"."Token" (
"id" SERIAL,
"createdAt" timestamp(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP,
"updatedAt" timestamp(3)  NOT NULL ,
"type" "TokenType" NOT NULL ,
"emailToken" text   ,
"valid" boolean  NOT NULL DEFAULT true,
"expiration" timestamp(3)  NOT NULL ,
"userId" integer  NOT NULL ,
PRIMARY KEY ("id"))

CREATE TABLE "public"."Course" (
"id" SERIAL,
"name" text  NOT NULL ,
"courseDetails" text   ,
PRIMARY KEY ("id"))

CREATE TABLE "public"."CourseEnrollment" (
"createdAt" timestamp(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP,
"role" "UserRole" NOT NULL ,
"userId" integer  NOT NULL ,
"courseId" integer  NOT NULL ,
PRIMARY KEY ("userId","courseId"))

CREATE TABLE "public"."Test" (
"id" SERIAL,
"updatedAt" timestamp(3)  NOT NULL ,
"name" text  NOT NULL ,
"date" timestamp(3)  NOT NULL ,
"courseId" integer  NOT NULL ,
PRIMARY KEY ("id"))

CREATE TABLE "public"."TestResult" (
"id" SERIAL,
"createdAt" timestamp(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP,
"result" integer  NOT NULL ,
"studentId" integer  NOT NULL ,
"graderId" integer  NOT NULL ,
"testId" integer  NOT NULL ,
PRIMARY KEY ("id"))

CREATE UNIQUE INDEX "User.email_unique" ON "public"."User"("email")

CREATE UNIQUE INDEX "Token.emailToken_unique" ON "public"."Token"("emailToken")

ALTER TABLE "public"."Token" ADD FOREIGN KEY ("userId")REFERENCES "public"."User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "public"."CourseEnrollment" ADD FOREIGN KEY ("userId")REFERENCES "public"."User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "public"."CourseEnrollment" ADD FOREIGN KEY ("courseId")REFERENCES "public"."Course"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "public"."Test" ADD FOREIGN KEY ("courseId")REFERENCES "public"."Course"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "public"."TestResult" ADD FOREIGN KEY ("studentId")REFERENCES "public"."User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "public"."TestResult" ADD FOREIGN KEY ("graderId")REFERENCES "public"."User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "public"."TestResult" ADD FOREIGN KEY ("testId")REFERENCES "public"."Test"("id") ON DELETE CASCADE ON UPDATE CASCADE
```

## Changes

```diff
diff --git schema.prisma schema.prisma
migration 20200730163012-init-db..20200821135747
--- datamodel.dml
+++ datamodel.dml
@@ -1,27 +1,43 @@
 generator client {
   provider        = "prisma-client-js"
-  previewFeatures = ["aggregateApi"]
+  previewFeatures = ["aggregateApi", "connectOrCreate"]
 }
 datasource db {
   provider = "postgresql"
-  url = "***"
+  url = "***"
 }
 model User {
-  id        Int    @default(autoincrement()) @id
-  email     String @unique
-  firstName String
-  lastName  String
+  id        Int     @default(autoincrement()) @id
+  email     String  @unique
+  firstName String?
+  lastName  String?
   social    Json?
   // Relation fields
   courses     CourseEnrollment[]
   testResults TestResult[]       @relation(name: "results")
   testsGraded TestResult[]       @relation(name: "graded")
+  tokens      Token[]
 }
+model Token {
+  id         Int       @default(autoincrement()) @id
+  createdAt  DateTime  @default(now())
+  updatedAt  DateTime  @updatedAt
+  type       TokenType
+  emailToken String?   @unique // Only used for short lived email tokens 
+  valid      Boolean   @default(true)
+  expiration DateTime
+
+  // Relation fields
+  user   User @relation(fields: [userId], references: [id])
+  userId Int
+
+}
+
 model Course {
   id            Int     @default(autoincrement()) @id
   name          String
   courseDetails String?
@@ -50,10 +66,10 @@
   name      String
   date      DateTime
   // Relation Fields
-  courseId   Int
-  course     Course       @relation(fields: [courseId], references: [id])
+  courseId    Int
+  course      Course       @relation(fields: [courseId], references: [id])
   testResults TestResult[]
 }
 model TestResult {
@@ -73,4 +89,9 @@
 enum UserRole {
   STUDENT
   TEACHER
 }
+
+enum TokenType {
+  EMAIL // used as a short lived token sent to the user's email
+  API
+}
```

