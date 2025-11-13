Explanation of Task 1

1. Design Approach (Pluggable Policies)
   I used the Strategy Pattern to achieve the pluggable eviction policy.
   Interface (ABC): I defined an abstract base class EvictionPolicy. This class acts as an interface, declaring three methods that the Cache class needs: on_access(key), on_put(key), and evict().
   Concrete Strategies: I created two "concrete strategy" classes that implement this interface: FIFOPolicy and LRUPolicy. Each one manages its own internal data structure to track item order.
 Context Class: The Cache class is the "context." It holds a reference to an instance of a policy object (e.g., self.policy = LRUPolicy()).
 Delegation: The Cache class's logic is simplified. It focuses on storing data in its self.data dictionary. When an event happens (like a get or a put), it delegates the work of tracking order to the policy object (e.g., self.policy.on_access(key)). When the cache is full, it simply asks the policy key_to_evict = self.policy.evict() and trusts that the policy will return the correct key based on its internal logic.
This design is highly extensible. To add a new Least Frequently Used (LFU) policy, we would simply create a new LFUPolicy class that implements the EvictionPolicy interface. No changes to the Cache class itself would be required.

2. Choice of Data Structures
  I used a combination of data structures for efficiency:
  Cache.data (Main Store): A standard Python dictionary (hash map), self.data = {}. This is used to store the actual key -> value pairs. This gives $O(1)$ average time complexity for get, put (updates), and deletions.
FIFOPolicy.queue: A collections.deque (a doubly-linked list). This is the perfect structure for a FIFO queue, providing $O(1)$ time for append() (for on_put) and popleft() (for evict).
LRUPolicy.order: A collections.OrderedDict. This is a powerful built-in Python structure that is essentially a hash map and a doubly-linked list combined. It is ideal for an LRU cache because it provides:$O(1)$ time for move_to_end() (used in on_access to mark an item as most recent).$O(1)$ time for popitem(last=False) (used in evict to remove the first or least recent item).$O(1)$ time for adding a new item (used in on_put).

3. Time Complexity
   Because of the data structures chosen, the time complexity for cache operations is highly efficient.
   get(key): $O(1)$ average
     It involves one dict lookup (self.data.get(key)), which is $O(1)$ average.
     It calls self.policy.on_access(). For FIFO, this is $O(1)$ (it does nothing). For LRU, this is OrderedDict.move_to_end(), which is also $O(1)$.
   put(key, value): $O(1)$ average
     Case 1 (Update existing key): One dict write ($O(1)$ avg) and one self.policy.on_access() call ($O(1)$). Total: $O(1)$ avg.
     Case 2 (Insert new key, no eviction): One dict write ($O(1)$ avg) and one self.policy.on_put() call (which is deque.append() or OrderedDict.__setitem__, both $O(1)$). Total: $O(1)$ avg.
     Case 3 (Insert new key, with eviction): One self.policy.evict() call (which is deque.popleft() or OrderedDict.popitem(), both $O(1)$), one dict delete ($O(1)$ avg), one dict write ($O(1)$ avg), and one self.policy.on_put() call ($O(1)$). Total: $O(1)$ avg.
All primary operations achieve the desired $O(1)$ average time complexity.

4. Edge Cases Considered
   get for non-existent key: The code checks if key not in self.data: and correctly returns None.
   put to update an existing key: The code correctly identifies this as an update, overwrites the value, and calls on_access to update its "recentness" (crucial for LRU).
   put to a full cache: The code correctly identifies len(self.data) >= self.capacity, calls evict() before adding the new item, deletes the old item, and then inserts the new one.
   Cache with capacity 1: The logic holds. Every new put will cause an eviction of the previous item.
   Invalid capacity: The constructor checks for capacity <= 0 and raises a ValueError to prevent invalid states.

Explanation of Task 2: Design Pattern Application

The most appropriate design pattern for this scenario is the Observer Pattern (also known as Publish-Subscribe).

1. Overview of the Pattern
The Observer Pattern defines a one-to-many dependency between objects. It consists of two main parts:

Subject (or Publisher): This is the object that holds the state or performs actions that other objects are interested in. In your case, this is the DocumentUploader. The Subject maintains a list of its "Observers."

Observer (or Subscriber): This is an interface (or abstract class) that defines a notification method (e.g., update()).

Concrete Observers: These are the actual objects that want to be notified (e.g., AnalyticsModule, AlertsModule). They implement the Observer interface.

When the Subject's state changes or an event occurs (a document is uploaded), it automatically notifies all its registered Observers by calling their update() method.

2. Application in this Context
Here is how the pattern would be applied to your notification system:

Key Classes and Interfaces:

IDocumentObserver (Interface):

This interface defines the contract for all subscribers.

Method: on_document_uploaded(document_details)

DocumentUploader (Subject):

This class contains the core upload logic.

Fields: private list_of_observers = []

Method: register(observer: IDocumentObserver)

Adds an observer to list_of_observers.

Method: unregister(observer: IDocumentObserver)

Removes an observer from list_of_observers.

Method: notify_observers(document_details)

Loops through list_of_observers and calls observer.on_document_uploaded(document_details) on each one.

Method: handle_upload(file)

Contains the logic to save the file.

After saving, it creates a document_details object (e.g., with file ID, path, user) and calls self.notify_observers(document_details).

Concrete Observers (Subscribers):

class AnalyticsModule implements IDocumentObserver:

on_document_uploaded(document_details): Starts its data processing job.

class AlertsModule implements IDocumentObserver:

on_document_uploaded(document_details): Checks if users need to be alerted and sends emails/messages.

class AuditLoggingModule implements IDocumentObserver:

on_document_uploaded(document_details): Writes a log entry to the audit database.

Interaction (Pseudo-code):

// --- Setup ---
// Create the main uploader (Subject)
uploader = new DocumentUploader()

// Create all modules (Observers)
analytics = new AnalyticsModule()
alerts = new AlertsModule()
audit = new AuditLoggingModule()

// --- Registration (Subscribing) ---
// The modules subscribe to the uploader
uploader.register(analytics)
uploader.register(alerts)
uploader.register(audit)

// --- Event Happens ---
// A user uploads a file, triggering the handle_upload method
uploader.handle_upload(some_file_object)

// --- Inside handle_upload() ---
// 1. File is saved
// 2. uploader.notify_observers(document_details) is called
// 3. This triggers:
//    - analytics.on_document_uploaded(...)
//    - alerts.on_document_uploaded(...)
//    - audit.on_document_uploaded(...)
3. How This Achieves Loose Coupling
This design is the definition of loose coupling. The DocumentUploader (Subject) is completely decoupled from the concrete modules (Observers).

No Hard-Coded References: The DocumentUploader does not know that AnalyticsModule, AlertsModule, or AuditLoggingModule even exist. It only knows that it has a list of objects that implement the IDocumentObserver interface.

Extensibility (Open/Closed Principle): The system is open for extension but closed for modification. If you need to add a new ReportGenerationModule tomorrow, you simply:

Create the new ReportGenerationModule class (implementing IDocumentObserver).

Register it with the uploader instance: uploader.register(new ReportGenerationModule()).

Crucially, you do not need to modify a single line of code in the DocumentUploader class.

Maintainability: You can add, remove, or temporarily disable modules (by unregistering them) at runtime without affecting the core upload logic or any of the other modules.   
   
   
   

   
     
   
     
   
   

  
