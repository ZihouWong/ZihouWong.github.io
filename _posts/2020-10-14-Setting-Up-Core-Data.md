---

title: "Setting Up Core Data"
date: 2020-11-14s
categories: 

layout: post
<!--comments: true-->

---

- Before you start setting up persistence, you should learn about the moving parts of Core Data, also known as the Core Data stack. The Core Data stack includes:

- A **managed object model** which defines model objects, called entities, and their relationships with other entities. Think of it as your database schema. In FaveFlicks, you’ll define the Movie entity as part of the managed object model inside FaveFlicks.xcdatamodeld. You’ll use the NSManagedObjectModel class to access your managed object model in the code.

- An **NSPersistentStoreCoordinator**, which manages the actual database.
- An **NSManagedObjectContext**, which is an in-memory scratchpad that lets you create, edit, delete or retrieve entities. Usually, you’ll work with a managed object context when you’re interacting with Core Data.
With that out of the way, it’s time to get started!