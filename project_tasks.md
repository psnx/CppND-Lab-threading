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

### Solution L2.2
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