# Udacity C++ Nanodegree program - Concurrency lab
---
:exclamation: Spoiler alert! This repo shows the solution to the tasks you are supposed to solve in the lab.
---  

[Here](https://github.com/udacity/CppND-Program-a-Concurrent-Traffic-Simulation) is the final project starter code.

# Installation
## Dependencies for Running Locally

- **cmake** >= 2.8
All OSes: click [here for installation instructions](https://cmake.org/install/)
- **make** >= 4.1 (Linux, Mac), 3.81 (Windows)
Linux: make is installed by default on most Linux distros
_Mac_: install Xcode command line tools to get make
_Windows_: Click here for installation instructions
OpenCV >= 4.1
- **The OpenCV 4.1.0** source code can be found [here](https://developer.apple.com/xcode/features/)
gcc/g++ >= 5.4
Linux: `gcc` / `g++` is installed by default on most Linux distros
Mac: same deal as make - install Xcode command line tools
Windows: recommend using [MinGW](http://www.mingw.org)

# Taks

## Task L2.1 
In method `Vehicle::drive()`, start up a task using `std::async` which takes a reference to the
method Intersection::addVehicleToQueue, the object _currDestination and a shared pointer to this using the
get_shared_this() function. Then, wait for the data to be available before proceeding to slow down.

### Solution L2.1:
```cpp
std::future<void> ftr = std::async(&Intersection::addVehicleToQueue, _currDestination, this->shared_from_this());
ftr.wait();
ftr.get();
```

## Task L2.2
In method `Intersection::addVehicleToQueue()`, add the new vehicle to the waiting line by
creating a promise, a corresponding future and then adding both to `_waitingVehicles`. Then wait until
the vehicle has been granted entry.

### Solution
```cpp
void Intersection::addVehicleToQueue(std::shared_ptr<Vehicle> vehicle)
{
    //std::cout removed everywhere
    std::promise<void> promise;
    std::future<void> future = promise.get_future();
    _waitingVehicles.pushBack(vehicle, std::move(promise));
    future.get();
}
```

## Task L2.3
In method `WaitingVehicles::permitEntryToFirstInQueue()`, get the entries from the
front of _promises and _vehicles. Then, fulfill promise and send signal back that permission to enter
has been granted. Finally, remove the front elements from both queues.  
### Solution
```cpp
// In Vehicle::drive()
//...
_promises.begin()->set_value();
_promises.erase(_promises.begin());
_vehicles.erase(_vehicles.begin());
```
## Task L3.1   
In class `WaitingVehicles`, safeguard all accesses to the private members `_vehicles` and `_promises` with an appropriate locking mechanism, that will not cause a deadlock situation where access to the resources is accidentally blocked.

### Solution  
Implement a `unique_lock<std::mutex>` in all pubvlic members where resources are accessed: 
```cpp
std::unique_lock<std::mutex> lck(mtx);
```

## Task L3.2  
Add a static mutex to the base class `TrafficObject` (called `_mtxCout`) and properly instantiate it in the source file. This mutex will be used in the next task to protect standard-out. 

### Solution  
```cpp
td::mutex TrafficObject::_mtxCout;
```

## Task L3.3  
In method `Intersection::addVehicleToQueue` and in `Vehicle::drive()` ensure that the text output locks the console as a shared resource. Use the mutex `_mtxCout` you have added to the base class TrafficObject in the previous task. Make sure that in between the two calls `to std::cout` at the beginning and at the end of addVehicleToQueue the lock is not held.

### Solution  
```cpp
void Intersection::addVehicleToQueue(std::shared_ptr<Vehicle> vehicle)
{
    std::unique_lock<std::mutex> lck(_mtxCout);
    std::cout << "Intersection #" << _id << "::addVehicleToQueue: thread id = " << std::this_thread::get_id() << std::endl;
    lck.unlock();
    std::promise<void> promise;
    std::future<void> future = promise.get_future();
    _waitingVehicles.pushBack(vehicle, std::move(promise));
    future.get();
    lck.lock();
    std::cout << "Intersection #" << _id << ": Vehicle #" << vehicle->getID() << " is granted entry." << std::endl;
}
```
