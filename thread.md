
## std::thread
first parameter is name of function to call, and rest of the parameters are arguments to the function.

    std::thread t1(func, std::ref(s));

#### Parameter passing
- All parameters are pass by value
- Use `std::ref` to pass by reference.
- function can be passed as function pointer, functors and lambda function.
- We can pass function as a `ref` in case of use with `functors` object.
	- `thread t1(std::ref(*functor), arg1, arg2);`
- `std::thread` starts thread as soon as it created.

#### Methods
 - t1.`join()` main process wait for the thread to complete.
  - t1.`detach()` → it will freely on its own. You cannot join detached
   thread.
  - t2.`joinable()` it will tell whether thread is joinable or not.

#### thread using lambda
```cpp
	int step =10;
    vector<int> partial_sum(5, 0);
    vector<thread> threads;
    for(int i =0 ; i < 5 ; i++) {
       threads.push_back([&partial_sum, int i, int step]() {
	       for (int j = i * step ; j < (i+1) * step ; j++) {
		       partial_sum[i] += j;
	       }
       });
    } 
    for(std::thread &th : threads) {
	    if (th.joinable()) {
		    th.join();
	    }
    }
    int result = std::accumulate(partial_sum.begin(), partial_sum.end(), 0);
    
```
### stt::async
- **Alternative** to `thread`, using this `class` we can capture return value of function using `future`
- Future : return value from `std::async` by calling `get()` function. it blocks main thread it future is not ready in similar manner to `join()`

```cpp
#include <thread>
#include <iostream>
#include <vector>
#include <future>
using namespace std;

int main() {
	
	vector<std::future<int> > tasks;
	int step = 10;
	for (int i =0 ; i < 5; i ++) {
		tasks.push_back(std::async([i , step]{
			int sum =0;
			for(int j = i*step ;j <  (i+1)* step; j++) {
				sum += j;
			}
			return sum;
		}));
	}
	
	int result =0;
	for(auto& t : tasks) {
		result+= t.get();	
	}
	cout <<result;
	return 0;
}
```
#### Avoid Data Race :
When two or more thread trying to write same variable then data racing condition arises. 
```cpp
    //! here thread t1 & t2 racing to access g_x global variable.
    auto t1 = std::thread([]() { g_x = 1; });
    auto t2 = std::thread([]() { g_x = 2; });
```
Compiler may change order of execution which may cause undefined behavior in multi-thread environment.
### Solution
1. `std::mutex` & Locks
2. `std::atomic`
3. Abstraction

## std::mutex
`Mutex` (mutual exclusion) is used to synchronize access to common resource with threads. using std::mutex;
`std::mutex g_mutex;`
- It also provide sequential consistency.

```cpp
void Incrementer() {
	for (size_t i = 0; i < 100; i++) {
		g_mutex.lock();
		g_counter++;
		g_mutex.unlock();
	}
}
```
#### Issues : 
 - lock and unlock should match 
 - If exception occurs after lock then that
   thread remains in locked state.

#### lock_guard (RAII)
Automatically unlocks while going out of scope.
```cpp
std::lock_guard<std::mutex> guard(g_mutex);
g_counter++;
```
#### unique_lock 
lock_guard + lock/unlock feature.
```cpp
std::unique_lock<std::mutex> guard(g_mutex);
g_counter++;
```
#### shared_lock 

 - It requires to have `std::shared_mutex` instead of `std::mutex` 
 - It allows multiple threads to have reading access if no write is
   happening in another thread. 
  - If thread is in writing critical section then it will not even allow reading thread.

#### Deadlock

```cpp
print1(string &s, int i) {
	std::lock_guard<mutex> guard1(mu1);
	std::lock_guard<mutex> guard2(mu2);
}

print2(string &s, int i) {
	std::lock_guard<mutex> guard2(mu2);
	std::lock_guard<mutex> guard1(mu1);
}

```

#### How to solve this deadlock

std::lock to lock more then one mutex simultaneously

#### multiple lock mechanism (`std::lock`)
```cpp
void Incrementer_Better() {
	for (size_t i = 0; i < 100; i++) {
		// lock both mutexes without deadlock
		std::lock(g_mutex1, g_mutex2);
		// make sure both already-locked mutexes are unlocked at the end of scope
		std::lock_guard<std::mutex> lock1(g_mutex1, std::adopt_lock);
		std::lock_guard<std::mutex> lock2(g_mutex2, std::adopt_lock);
		g_counter++;
	}
}
```
#### `std::scoped_lock` : c++17 syntax with RAII
`std::scoped_lock scoped_lock_name(g_mutex1, g_mutex2);`


#### Fine Grained Locking system

- std::unique_lock<mutex> locker(mu);

- it provide flexibility to lock it and unlock it.

- std::unique_lock<mutex> locker(mu, std::defer); // do not lock mutex mu

- `lock_guard` does not provide this flexiblity.

- `unique_lock` can be moved from one lock to another:

`std::unique_lock<mutex> locker2 = sd::move(locker);`

#### Lazy initialization :

```cpp
std::once_flag _flag;
std::call_once (_flag, & { f.open(”a.txt”)};
```

## Conditional Variable
`std::condition_variable g_cv;`
- Protect conditional variable using mutex.
- Always use predicate.

### Usage
We use conditional variable for thread communication and message passing.

---
`wait` does two things :
- if mutex is locked then it unblocks it so that other thread can continue with their work.
- if mutex is unlocked then it blocks it so that this current thread proceed with its work.

Important point to note in below producer - consumer program is that condition or `wait` is protected by `unique_lock` and `notify_one` is not under critical section.


```cpp
std::mutex g_mutex;
std::condition_variable g_cv;
bool g_ready = false;
int g_data = 0;
void ConsumeData(int& data) {}
void Consumer() {
	int data = 0;
	for (int i = 0; i < 100; i++) {
		std::unique_lock<std::mutex> ul(g_mutex);
		//! critical section starts
		// if blocked, ul.unlock() is automatically called.
		// if unblocked, ul.lock() is automatically called.
		g_cv.wait(ul, []() { return g_ready; });
		// Sample data
		data = g_data;
		std::cout << "data: " << data << std::endl;
		g_ready = false;
		ul.unlock();
		//critical section end
		g_cv.notify_one();
		ConsumeData(data);
	}
}

void Producer() {
	for (int i = 0; i < 100; i++) {
		std::unique_lock<std::mutex> ul(g_mutex);
		// Produce data
		// critical section starts
		g_data = GenRandomValue();
		std::this_thread::sleep_for(std::chrono::milliseconds(100));
		g_ready = true;
		ul.unlock();
		// critical section end here.
		g_cv.notify_one();
		ul.lock();
		//critical section starts ans wait is inside critical section.
		g_cv.wait(ul, []() { return g_ready == false; });
	}
}

int main() {
	std::thread t1(Consumer);
	std::thread t2(Producer);
	t1.join();
	t2.join();
	return 0;
}
```

## std::atomic
`std::atomic <int> x(0);`
it ensures read, increment and write atomically. 

#### What type are allowed 
 - integral types such as `int` `unsigned` , `bool` floating types,
 - `float`, `U*`(pointer) and `std::shared_ptr`

#### User Defined Atomic:
user defined types can be atomic but these classes should be trivially copyable. 

##### It should satisfy following conditions :

- continous block of memory so that it can be copy using memcpy
- no user defiend copy, assignment and move constructor
- no virtual function or virtual base class.

`auto is_trivialy  = std::is_trivially_copyable<A>::value;`

`std::atmoic` is not copy assignable. one atomic variable cannot be assigned to another atomic variable. 
### memeber function of atomic
#### operator=
```cpp
std::atmoic<int> a(10);
int b;
b = a;
a = b;
```
#### store & load
```cpp
std::atomic<int> a(10);
int b;
b = a.load(); //equivalent to operator=
a.store(b);
```

#### exchange
`old_value = atomic_x.exchange(new_value)`
equivalent to 
```
vzold_value = atomic_x;
atomic_x = new_value;
```

#### compare_exchange_strong
`bool success = atomic_x.compare_exchange_strong(expected, desired);`
equivalent to 
```cpp
//Atomically do this: 
if (atomic_x == expected) {
	atomic_x = desired; 
	return true; 
}
else { 
	expected = atomic_x;
	return false;
}
```
#### Why do we need compare_exchange?
Lets say if we want to calculate : 
`atomix_x = f(atomic_x)`

while calulating f(x), we check if someone else change value of x, so use new value of x and recalculate f(x) like below :
```cpp
auto old_x = atomic_x.load();
while(!atomic_x.compare_exchange_strong(old_x, f(old_x));
```
compare_exchange_weak, similar to `compare_exchange_strong` only difference is that there may be spurious success in case of `weak`.

### specialize memeber function

memory_order_seq_cst
A sql_cst store synchronize with a sql_cst load on same variable.
it keeps total order on the threads. 
It provides synchronization and total order.
