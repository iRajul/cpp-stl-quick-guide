
## Container Adapter
  ### priority_queue : 
	  template< class T, class Container = std::vector, class Compare = std::less > class priority_queue

 - `vector` and `deque` can be used as container.  
 - By default it is max
   heap, can be converted into min heap using `greater` comparator.
 - `push()` -> Effectively calls c.push_back(value);
   std::push_heap(c.begin(), c.end(), comp);  
 - `pop()` ->
   std::pop_heap(c.begin(), c.end(), comp); c.pop_back();
