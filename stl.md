> Made this guide as a notes after reading Effective STL by scott meyers


### CONTAINERS

### Item 1. Choose your containers with care.
1) The standard STL sequence containers, vector, string, deque, and list.
2) The standard STL associative containers, set, multiset, map and multimap.
3) `vector<char>` as a replacement for `string`. (Item 13)
4) vector as a replacement for the standard associative containers. (Item 23)

* Contiguous-memory containers - vector, string, deque (random access iterators)
* Node-based containers store only a single element per chunk : list + all associative continers.

Do you need to minimize iterator, pointer, and reference invalidation? If so, you'll want to use node-based containers.

---

### Item 2. Beware the illusion of container-independent code.
We strive to write container-independent code. Like writing code using vector 
but still preserve the option of replacing it with something like a deque or a list later.

This should be avoid bcoz many methods are avialable for only one type of containers.
Insert signature is different for sequence and associative containers.
The different containers are different, and they have strengths and 
weaknesses that vary in significant ways. They're not designed to be interchangeable,

-----------------------------------------------------
### Item 3. Make copying cheap and correct for objects in containers.
When you insert or get object from stl container they return copy of object. Copy in, copy out.

* If you have user-defined object that has heavy copy operation, then think about it.
* Objec Slicing : if you create a container of base class objects and you try to insert derived class objects into it, the derived-ness of the objects will be removed as the objects are copied (via the base class copy constructor) into the container.

An easy way to make copying efficient, correct, and immune to the slicing problem is to create containers of pointers instead of containers of objects.

------------------------------------------------------

### Item 4. Call empty() instead of checking size() against zero.
* `empty` is a constant-time operation for all standard containers, but for some 
list implementations, `size` takes linear time.

---

### Item 5. Prefer range member functions to their single-element counterparts.

Given two vectors, v1 and v2, what's the easiest way to make v1’s contents be the same as the second half of v2's?
```
// assign range flavour
v1.assign(v2.begin() + v2.size() /2, v2.end()); 
// insert range flavour
v1.insert(v1 .end(), v2.begin() + v2.size() / 2, v2.end()); 
```
why use range member function?
* It's generally less work to write the code using the range member functions.
* Range member functions tend to lead to code that is clearer and more straightforward.
* using range leads to one function call and using loop leads to n-1 extra function calls.
* they are more efficient than regular loops.

Range construction:  All standard containers offer a constructor of this form:
`container::container( Inputlterator begin, Inputlterator end);`

Range insertion :
1) Sequence Container:
`void container::insert(iterator position, Inputlterator begin, InputIterator end);`
2) Associative container:
`void container::insert(lnputIterator begin, Inputlterator end);`
Range erasure: 
1) Sequence containers :
`iterator container::erase(iterator begin, iterator end);`
2) Associative containers :
`void container::erase(iterator begin, iterator end);`
Range assignment: 
1) Sequence Container:
`void container::assign(lnputIterator begin, Inputlterator end);`

-----------------------------------------------------

### Item 6. Be alert for C++'s most vexing parse.
TODO

-----------------------------------------------------

### Item 7. When using containers of newed pointers, remember to delete the 
pointers before the container is destroyed.

* Either delete all elements of container using loop or use smart pointer to store elements.

-----------------------------------------------------

### Item 8. Never create containers of auto_ptrs.
* `auto_ptr` transfer their ownership to caller. so never use them!!

-----------------------------------------------------

### Item 9. Choose carefully among erasing options.

1) Get rid of all the objects in c with the value 1963
* For String, vector, deque
`c.erase( remove(c.begin(), c.end(), 1963), c.end()); // the erase-remove idiom`

* For List 
`c.remove(1963);`

* For Associative containers:
`c.erase(1963);`
>> the associative container erase member function has the advantage of being based on equivalence instead of equality,  a distinction whose importance is explained in Item 19.

2) Eliminate every object for which the following predicate (see Item 39) returns true:
* For vector, string, deque
`c.erase(remove_if(c.begin(), c.end(), badValue), c.end());`

* For List 
`c.remove_if(badValue);`

* For associative containers:
* >> Less Efficient:
`remove_copy_if(c.begin(), c.end(), inserter( goodValues, goodValues.end()), badValue):`
// copy unremoved values from c to goodValues
c.swap(goodValues):

* More efficient: 
```
AssocContainer<int> c;
for (AssocContainer<int>::iterator i = c.begin(); i != c.end(); ){
	if (badValue(*I)) c.erase(i++); 
	else ++i; 
}
//the 3rd part of the for loop is empty; i is now incremented below for bad values, pass the current i to erase and increment i as a side effect; for good values, just increment i
```
To do something inside the loop (in addition to erasing objects):
* If the container is a standard sequence container, write a loop to walk the container elements, being sure to update your iterator with erase's return value each time von call it.
* If the container is a standard associative container, write a loop to walk the container elements, being sure to postincrement your iterator when you pass it to erase.

---

### Item 10
TODO

---

### Item 11
TODO

---

### Item 12
TODO

---


### STRING AND VECTOR

### Item 13. Prefer vector and string to dynamically allocated arrays.
string implementation may involve refernce counting so must see their document or use `vector<char>`

---
### Item 14. Use reserve to avoid unnecessary reallocations.

```
vector<int> v;
v.reserve(1000);
for (int i = 1; i <= 1000; ++i) v.push_back(i);
```
This should result in zero reallocations during the loop.

Reallocation in vector invalidate iterators, pointer and references

There are two common ways to use reserve to avoid unneeded reallocations. 
* The first is applicable when you know exactly or approximately how many elements will ultimately end up in your container. 
In that case, as in the vector code above, you simply reserve the appropriate amount of space in advance. 

* The second way is to reserve the maximum space you could ever need. then, once you've added all your data, trim off any excess capacity.

--------------------------------------------------------

### Item 15. Be aware of variations in string implementations. 
Different vendors can have different implementation of string.
string objects may range in size from one to at least seven times the size of char* pointers.

--------------------------------------------------------

### Item 16. Know how to pass vector and string data to legacy APIs.
For vector:
```
if (!v.empty()) {
doSomething(&v[0], v.size());
}
```
>> Beware : Dont use v.begin() in place of &v[0]

For string:
`c_str()`

The approach to getting a pointer to container data that works for vectors isn't reliable for strings, because 
1) the data for strings are not guaranteed to be stored in contiguous memory, and 
2) the internal representation of a string is not guaranteed to end with a null character.

----------------------------------------------------------

### Item 17. Use "the swap trick" to trim excess capacity
//shrink to fit
```
vector<Contestant>(contestants).swap(contestants); 
string(s).swap(s);
```
To clear container : 
```
vector<Contestant> v;
string s;
vector<Contestant>().swap(v); //clear v and minimize its capacity
string().swap(s); // clear s and minimize its capacity
```
---

### Item 18. Avoid using vector<bool>:
Two things wrong with `vector<bool>`. 
First, it's not an STL container. 
Second, it doesn't hold bools.
```
vector<bool> v;
bool *pb = &v[0]; // error! the expression on the right is of type vector<bool>::reference*, not bool*
```
Because it won't compile, `vector<bool>` fails to satisfy the requirements for STL containers.

Alternative to use : 
* `deque<bool> // it would not allocate in contiguous manner like vector.`
* `bitset`

---

### ASSOCIATIVE CONTAINER

### Item 19. Understand the difference between equality and equivalence.
Equality, which is based on `operator==`
Equivalence is based on the relative ordering of object values in a sorted range.
`!c.key_comp()(x, y) && !c.key_comp()(y, x)`
The standard associative containers are kept in sorted order, so each container must have a comparison function (less, by default) that defines how to keep things sorted.

---

### Item 20. Specify comparison types for associative containers of pointers.
Anytime you create associative containers of pointers, figure you're probably going to have to specify the container's comparison type, too. 
Most of the time, your comparison type will just dereference the pointers and compare the pointed-to objects.
```
struct DereferenceLess {
template <typename PtrType>
bool operator()(PtrType pT1, // parameters are passed by
		PtrType pT2) const // value, because we expect them
	{   // to be (or to act like) pointers
		return *pT1 < *pT2;
	}
};
```

Usage : `set<string*, DereferenceLess> ssp;`

---------------------------------------------------------

### Item 21. Always have comparison functions return false for equal values.

---------------------------------------------------------

### Item 22. Avoid in-place key modification in set and multiset.

Key is const in map and multimap but it is non-const in set and multi-set but some vendors provide const key in set too. 

Dont modify key in set as it can corrupt sorting order of keys in set.
If you want to modify content of key which is not  a part of comparator then do it like this: 
```
if (i != se.end()) { // cast away
	const_cast<Employee&>(*i).setTitle("Corporate Deity"); //constness
}
```
Use "Employee&" not "Employee" because by using "Employee" it will modify temporary object not actual object.

Or 
Follow five step process:
```
EmplDSet::iterator i =
se.find(selectedlD); // Step 1: find element to change

if(i!=se.end()){
	Employee e(*i); // Step 2: copy the element
	se.erase(i++); // Step 3: remove the element; increment the iterator to maintain its validity (see Item 9)
	e.setTitle("Corporate Deity"); // Step 4: modify the copy
	se.insert(i, e); // Step 5: insert new value; hint that its location is the same as that of the
}
```
------------------------------------------------------------

### Item 23. Consider replacing associative containers with sorted vectors.

Assuming our data structures are big enough, they'll be split across multiple memory pages, but the vector will require fewer pages than the associative container.
Bottom Line : 
Storing data in a sorted vector is likely to consume less memory than storing the same data in a standard associative container, and searching a sorted vector via binary search is likely to be faster than searching a standard associative container when page faults are taken into account.
You need to write comparator for `pair<key,value>` if you choose vector instead of associative container.

When to use sorted vector over associative containers:
your program uses the data structure in the phased manner that is first only insertions and then only lookups
It makes sense to consider using a sorted vector instead of an associative container only when you know that your data structure is used in such a way that lookups are almost never mixed with insertions and erasures because inserting on sorted vector can be expensive operation.

-------------------------------------------------------------

### Item 24. Choose carefully between map::operator[] and map-insert when efficiency is important
when an "add" is performed, map-insert saves you three function calls: 
1) one to create a temporary default-constructed Widget object, 
2) one to destruct that temporary object, and 
3) one to Widget's assignment operator.

`operator[]` is preferable when updating the value of an element that's already in the map.	
`m[k] = v; // use operator[] to update k's value to be v`

`m.insert(
	IntWidgetMap::value_type(k, v)).first->second = v; // use insert to update k's value to be v`

-------------------------------------------------------------

### Item 25 : hash_map, hash_set 

TODO

--------------------------------------------------------------

### ITERATORS

### Item 26. Prefer iterator to const iterator, reverse_iterator, and const_reverse_iterator.

1) Some versions of insert and erase require iterators. 
* If you want to call those functions, you're going to have to produce iterators, const and reverse iterators won't do.
** Example :
** iterator insert(iterator position, const T& x);
** iterator erase(iterator position);
** iterator erase(iterator rangeBegin, iterator rangeEnd);
	
2) It's not possible to implicitly convert a const iterator to an iterator, and 
	the technique described in Item 27 for generating an iterator from a const_iterator is neither universally applicable nor guaranteed to be efficient.

3) Conversion from a reverse_iterator to an iterator may require iterator adjustment after the conversion. Item 28 explains when and why.

4) comparison of const iterator with non-const iterator would not compile.
* if (ci == i) // not compile.
* if (i == ci) // It works though 

--------------------------------------------------------------

### Item 27. Use distance and advance to convert a container's const_iterators to iterators.

`Iter i(const_cast<lter>(ci)); ` // It would not compile because const_iterator and iterator are of different type except for string and vector.

Make a non-const iterator and move it to where const iterator is pointing.
```
Iter i(d.begin()); // initialize i to d.begin() 
advance(i, distance<ConstIter>(i, ci));//figure the distance between i and ci (as const_iterators), then move i that distance.
```
We need to specify "ConstIter" while calling distance because type of argument for distance function should be same.

---

### Item 28. Understand how to use a reverse_iterator's base iterator.

Lets say there is vector of 5 element
```
|--------------------
| 1 | 2 | 3 | 4 | 5 |
|--------------------
```

```vector<int>::reverse_iterator ri = // make ri point to the 3
find(v.rbegin(), v.rend(), 3);
vector<int>::iterator i(ri.base());// it points to 4
```

1) For insertion : using reverse_iterator we insert using ri.base().
```
|-------------------------
| 1 | 2 | 3 | 99 | 4 | 5 |
|-------------------------
```
Example  : Insert 99 at position of 3 from right to left.
`ri.begin would do the trick.`

2) For deletion : Delete the element pointed by ri.
`v.erase(++ri).base()); // erase the element pointed to by ri; this should always compile.`

---------------------------------------------------------------

### Item 29 : TODO

---------------------------------------------------------------

### ALGORITHMS

### Item 30. Make sure destination ranges are big enough.

Transform:
```
vector<int> results; // apply transmogrify to
transform(values.begin(), values.end(), //each object in values,
back_inserter(results), //inserting the return
transmogrify);
```
Use reserve to save time in allocation using:
`results.reserve(results.size() + values.size());`

Then use back_inserter, front_inserter , inserter.

If enough space is there in result and you want to overwrite result than you can use:
```
transform(values.begin(), values.end(), // overwrite the first
results.begin(), // values.size() elements of
transmogrify);
```

-----------------------------------------------------------------

### Item 31. Know your sorting options.

1) `partial_sort`:(Not stable)
Rearranges the elements in the range [first,last), in such a way that the
elements before middle are the smallest elements in the entire range and 
are sorted in ascending order, while the remaining elements are left 
without any specific order.

`partial_sort (widgets.begin(), widgets.begin() + 20, widgets.end(), qualityCompare);`
// put the best 20 elements (in order) at the front of widgets.

If all you care about is that the 20 best but don't care order of 20 element than use nth_element.
2) `nth_element`:(similar to partition algo)(Not Stable)
nth_element is a partial sorting algorithm that rearranges elements in [first, last) such that:
The element pointed at by nth is changed to whatever element would occur in that position if [first, last) was sorted.
All of the elements before this new nth element are less than or equal to the elements after the new nth element.

nth_element (widgets.begin(), widgets.begin() + 20, widgets.end(), qualityCompare); 
// put the best 20 elements at the front of widgets, but don't worry about their order.

3) For stable sorting use `stable_sort`

4) `partition` and `stable_partition`
Partition range in two
Rearranges the elements from the range [first,last), in such a way that all the elements 
for which pred returns true precede all those for which it returns false. The iterator 
returned points to the first element of the second group.
`vector<Widget>::iterator goodEnd = partition(widgets.begin(), widgets.end(), hasAcceptableQuality);`
// move all widgets satisfying hasAcceptableQuality to the front of widgets, 
// and return an iterator to the first widget that isn't satisfactory.

------------------------------------------------------------------

### Item 32. Follow remove-like algorithms by erase if you really want to remove something.

