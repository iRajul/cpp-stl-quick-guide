
### std::thread
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

### std::mutex
`Mutex` (mutual exclusion) is used to synchronize access to common resource with threads. using std::mutex;

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

    std::unique_lock<std::mutex> guard(g_mutex);
    g_counter++;

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
