theme: Merriweather, 4

#CoreData

---

#What is CoreData?

- Handles persistance for complex object graphs.
- Backed by ObjC runtime.
- De facto ORM for iOS and macOS

---

> Contrary to popular belief, Core Data is not an 
Object-Relational Mapper, but rather an object graph
and persistence framework, capable of [...]. Using
Core Data as an ORM necessarily limits the 
capabilities of Core Data and muddies its conceptual
purity. 

*NSHipster.com*  

---

#Backend

- `NSBinaryStoreType`
- `NSInMemoryStoreType`
- `NSSQLiteStoreType`


---

#Features

- Object uniquing
- Lazy load or *faulting*
- Migrations from database schema
	- Mostly automatic
- `NSPredicate`
- Change tracking via KVO
- Undo/Redo	 

--- 

#Stack

- This is what you need to create and hook to use the database.
- It's fairly complex
	- Because of it's flexibility.
	- Optimal stack setup depends on the use case.

--- 

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/UmW4mI2.png)

---

##NSManagedObjectModel

- Describes the relationships between your data model objects.

- Entitites are represented by `NSManagedObject`s
- Relationship between entities:
	- 1:1
	- 1:N
	- N:N

---

![left fit](https://i.imgur.com/54IwgrB.png)

###`Model.xcdatamodeld` -> 🔂 -> `Model.momd`

---

##NSManagedObject

- Represents the basic data entities.
	- `GIF`
- Is not used directly, but rather subclassed.
	- `ManagedGIF`
- Backed by ObjC runtime
	- `@NSManaged` attributes 
- Can be *faulted* if neccesary

---

##NSManagedObject

- Contains properties, relationships and helper methods to modify these.
	- Only mody relationships via the *accessors* ⚠️ 

- Relationships are unordered by default.

---

[.background-color: #FFFFFF]
![left fit](https://i.imgur.com/RguTcak.png)
![right fit](https://i.imgur.com/JXYj0ck.png)

---

##NSManagedObjectContext

- Stores a object graph from the store
- Only contains *some* objects from the store, either:
	- Fully fetched.
	- Faulted.
- Not thread-safe. 
	- One context per-thread

---

##NSManagedObjectContext

- Using it you'll do operations on objects:
	- Creation
	- Erase 
	- Saving
	- Search
	- Undo/Redo

---

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/veDBpIJ.png)

---

##Stack kinds:

- Unique context
- Nested contexts
- Sibling contexts
- Hybrid contexts

---

##Unique contexts

- Simple but low performance
	- One context to read write
	- It'll block the main thread 

---

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/XOmtyBG.png)

---

##Nested contexts

- Compromise between simplicity and performance.
- Imports are performed in a background thread
- Reads are perfomed in main thread
	- Automatic sync

---
 
[.background-color: #FFFFFF]
![fit](https://i.imgur.com/RXJXQRn.png)

---

##Sibling contexts

- Improved performance.
- Contexts are indipendent from each other.
	- Thread-wise they are also indipendent.
- Requires manual synchronization via the `NSManagedObjectContextDidSaveNotification`.
	- This will block the main context. 

---

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/LNXNrp2.png)

---

##Hybrid Stack

- Adds a parent context for saving in a background thread.
- Then, a child context will be bound to the main thread.
- Saving is performed using transient contexts, also child of the main one.
- ⚠️ It's the most complex one, since race conditions may occur.
	- GCD queues to guard against these are recommended

---

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/vOnH72u.png)

---

##NSFetchRequest

- The equivalent of an SQL statement.
- Searches, creates, filters and limits the objects within a CoreData context.
- High computational cost, may be async.

---

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/sYnbMbw.png)

---

##NSFetchedResultsController

- Observes the changes within an `NSFetchRequest` in a *reactive* way.
- Conceived for integrating with a `UITableView`/`UICollectionView` through a delegate
- Listens for changes in objects within a context with `NSManagedObjectContextObjectsDidChangeNotification`.

---

[.background-color: #FFFFFF]
![fit](https://i.imgur.com/23Ml8fs.png)
