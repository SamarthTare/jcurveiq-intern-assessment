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
   
   
   

   
     
   
     
   
   

  
