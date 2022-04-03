---
title: "Callable Stoppable Objects in C++"
author: "Kaushal Kishore"
layout: "post"
categories: cpp
date: 2022-04-02
---

Sometimes you want to write deploy some background tasks which can be stopped on-demand. In this blog we will discuss a code-snippet which defines a callable-stoppable object using C++17.

We will use `std::promise<T>` and `std::future<T>`. We can understand `std::promise<T>` as a contract which will be fulfilled anytime later during the program execution. 

`std::promise<T>::set_value()` is used to fulfill that contract. The terms of the contract is: **set the value in the shared state** and **make the state ready**.  The shared state can later be accessed by the `std::future<T>::get()` which will wait until the status is set ready by the promise object.

The following image summarize that:

![Promise Future](/img/cpp/callable-objects/promise_future.png)

For this task we don't need to retrieve the value set by promise object, we are interested in the state. Whenever the state is ready, it means stop is requested.

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <functional>
#include <chrono>
#include <array>
#include <memory>

using std::chrono::milliseconds;

class CallableStoppableTask {
    std::promise<void>  exit_signal;    // shared state + shared mem
    std::future<void>   future_obj;     // read shared state

public:

    CallableStoppableTask() : future_obj(exit_signal.get_future()) {}

    CallableStoppableTask(const CallableStoppableTask&) = delete;
    CallableStoppableTask& operator=(const CallableStoppableTask&) = delete;

    virtual void task() = 0;

    void run() final {
        while(true){
            task();
            if (is_stop_requested()) break;
        }
    };

    void operator()() final { run(); }

    bool is_stop_requested(int timeout_milliseconds = 0) const {
        return (future_obj.wait_for(milliseconds(timeout_milliseconds)) == std::future_status::ready);
    }

    void request_stop() { exit_signal.set_value(); }
};
```

## Example

Defining a sample task:

```cpp
struct Car {
    int     model;
    int     price;

    explicit Car(const int& arg_model = -1, const int& arg_price = -1)
    : model(arg_model), price(arg_price) {}
};

class CallableSampleTask : public CallableStoppableTask {
    
    CallableSampleTask(const CallableSampleTask&) = delete;

    CallableSampleTask& operator=(const CallableSampleTask&) = delete;

    void task() override {
        std::cout << "Running Some Car Sample Task.. " << std::endl;
    }

public:
    CallableSampleTask(const Car& arg_car = Car()) {} 
};
```

Tasks in action:

```cpp
#define NUM_TASK 5

static std::array<std::unique_ptr<CallableSampleTask>, NUM_TASK> ar_uptr_task;

int main() {
    // declare some tasks
    for (int i = 0; i < NUM_TASK; i++) {
        ar_uptr_task[i] = std::make_unique<CallableSampleTask>();
        auto discard_ret = std::async(std::launch::async, std::ref(*ar_uptr_task[i]));
    }

    // can be later stopped up like this
     for (int i = 0; i < NUM_TASK; i++) {
        ar_uptr_task[i]->request_stop();
    }

    return 0;
}
```

## References

* [`std::promise`](https://en.cppreference.com/w/cpp/thread/promise)
* [`std::future`](https://en.cppreference.com/w/cpp/thread/future)