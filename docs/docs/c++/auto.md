## auto

**auto** is a new keyword introduced in C++11. It is used to automatically deduce the type of a variable from its initializer. It is mostly used in generic programming when the type of the variable is dependent on the type of the initializer. It is also used in range-based for loop to automatically deduce the data type of the loop variable.

Let's say we have a vector of integers and we want to iterate over it.
```
# auto.cpp
#include <vector>
std::vector<int> v = {1,2,3,4,5};
```
Without **auto**, a typical way to iterate over a vector looks like this:
```
# auto.cpp
for(std::vector<int>::iterator it = v.begin(); it < v.end(); it++){
        std::cout << *it << std::endl;
    }
```
which is quite verbose. With **auto**, we can write it like this:
```
# auto.cpp
for(auto it = v.begin(); it < v.end(); it++){
        std::cout << *it << std::endl;
    }
```
or we can even use a new syntax introduced in C++11 called **range-based for loop**:
```
# auto.cpp
for(auto i : v){
        std::cout << i << std::endl;
    }
```

### **Some rules of auto**
```
int i = 10;
auto a = i, &b = i, *c = &i; // a is int, b is int&, c is int*
auto d = 0, f = 1.0;         // error，since d is int while f is double, compiler can't deduce
auto e;                      // error，auto can't deduce the type of uninitialized variable
```
```
void func(auto value) {}     // error，auto can't deduce the type of function parameters

class A {
    auto a = 1;              // error，auto not allowed in non-static class member
    static const auto b = 1; // ok, b is int
};

void func2() {
    int a[10] = {0};
    auto b = a;              // ok, b is int*
    auto c[10] = a;          // error，auto cannot deduce array type but can deduce pointer type
    vector<int> d;
    vector<auto> f = d;      // error，auto can't deduce template parameter
}
```
- auto can't deduce the type of function parameters
- auto can't be used in non-static class member
- auto can't deduce array type but can deduce pointer type
- auto can't deduce template parameter
- auto can't deduce the type of uninitialized variable
- auto can't deduce different types of variables in one statement
### **auto and _const & volatile_**
```
int i = 0;
auto *a = &i;           // a is int*
auto &b = i;            // b is int&
auto c = b;             // c is int, reference is ignored
const auto d = i;       // d is const int
auto e = d;             // e is int
const auto& f = e;      // f is onst int&
auto &g = f;            // g is const int&
```
- while not declared as refernce or pointer, auto will ignore the **const** and **volatile** qualifier
- vice versa
### **When to use *auto*?**
There is no absulote answer to this question. But if utlizing **auto** makes the code cleaner and more readable, why not?
```
auto func = [&] {   // we don't care what's the return type of this lambda function anyway...
    cout << "xxx";
}; 
auto asyncfunc = std::async(std::launch::async, func);
// as we are too lazy to memorize what the return type is, this is the perfect spot for auto
```