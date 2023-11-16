## Smart Pointer in C++
*Last update: Nov 15, 2023*
### Just some boring complaints
Recently I am thriving to write my own web server library in C++, and as always, got many pointers involved, as well as the error like this:
```
Segmentation fault
```
From time to time I saw this I wanted to smash my laptop! Therefore I turn to smart pointer for help.

### Problems with naked pointers(is that how it called?)
- Memory leak
- Wild pointer
- Dangling pointer
- Segmentation fault (you can't touch there my man)

### Header file required
```cpp
#include <memory>
```
***
### auto_ptr
**auto_ptr** is the first smart pointer template introduced in C++98. The way we use it is similar to those who will be introduced later.
```cpp
// point to a vector<int>
auto_ptr< vector<int> > ptr(new vector<int>());
cout << ptr->size() << endl;
```
As **auto_ptr** is no longer used after C++11, I am not gonna talk a lot on its feature. Just describe you the big picture of how it can be **smart**.

In fact, **auto_ptr** is wrapping the so called **naked pointer** into an object, with **operator\*** and **->** overloaded, so that it can behave just like a naked pointer.

There are three common methods used for smart pointers:
1. **get()**  returns the address of the pointer it saves
```cpp
auto_ptr< vector<int> > ptr = new vector<int>();
cout << ptr.get() << endl;
```

2. **release()** decouples the smart pointer object with the saved pointer and return the naked pointer
```cpp
auto_ptr< vector<int> > ptr(new vector<int>());
vector<int>* origin = ptr.release();
delete origin;
// you have to delete it manually after release()
// auto_ptr no longer manages this pointer
```

3. **reset()** can take another pointer as argument(optional), and free the pointer it manages if they are different
```cpp
auto_ptr< vector<int> > ptr(new vector<int>());
ptr.reset();   // delete the pointer it saves
vector<int>* new_ptr = new vector<int>();
ptr.reset(new_ptr);    // ptr now saves new_ptr
```

#### Suggestion
don't declare a smart pointer as a global variable or as a pointer, it's meaningless
```cpp
auto_ptr<Test> *tp = new auto_ptr<Test>(new Test);  // don't do this
```
Note that **auto_ptr** is replaced by **unique_ptr** in C++11. Because
1. copying or assigning a smart pointer to another will change the ownership of the resource.
```cpp
auto_ptr<string> p1(new string("Hi!"));
auto_ptr<string> p2(new string("I'm Josh."));
cout << "p1：" << p1.get() << endl; // 012A8750
cout << "p2：" << p2.get() << endl; // 012A8510
  
p1 = p2;	// assigning p2 to p1 leads to the loss of ownership to the resource of p2
  
cout << "after p1 = p2 :" << endl;
cout << "p1：" << p1.get() << endl; // 012A8510
cout << "p2：" << p2.get() << endl; // 00000000
```
2. Using **auto_ptr** for STL is risky, since it requires copying and assigning value
```cpp
vector<auto_ptr<string>> vec;
auto_ptr<string> p3(new string("I'm P3"));
auto_ptr<string> p4(new string("I'm P4"));
  
// have to use move semantic before pushing back to the container
vec.push_back(std::move(p3));
vec.push_back(std::move(p4));
  
cout << "vec.at(0)：" <<  *vec.at(0) << endl;
cout << "vec[1]：" <<  *vec[1] << endl;
  
// here comes the risk
vec[0] = vec[1];	// Assigning!
cout << "vec.at(0)：" << *vec.at(0) << endl; // trying to access NULL
cout << "vec[1]：" << *vec[1] << endl;
```
3. not supporting memory management for array
```cpp
auto_ptr<int[]> array(new int[5]);  // can't do this
```
***
### unique_ptr
**unique_ptr** is almost the same as **auto_ptr**, except that:
1. **exclusive ownership**: two pointers can't point to the same object
2. not allows **left-value** copy construction or assigning, but allows temporary **right-value** move construction or assigning
3. supports array memory management
```cpp
p1 = p2;                                // ban left-value assigning
unique_ptr<string> p3(p2);              // ban left-value copy construction
unique_ptr<string> p3(std::move(p1));   // allow
p1 = std::move(p2);                     // allow, p2 releases its resource
unique_ptr<int[]> array(new int[5]);    // allow
``` 

#### release memory actively
```cpp
unique_ptr<Test> t(new Test);
// any of these works
t = NULL;
t = nullptr;
t.reset();
```

#### transfer ownership
```cpp
Test* t1 = t.release();
```

#### reset
```cpp
t.reset(new Test);
```

#### Be careful
Just like **monogamy**, **unique_ptr** don't share one pointer they save. Look at this example:
```cpp
Wife* w1 = new Wife();
unique_ptr<Wife> husband(w1);   // husband now manages w1
unique_ptr<Wife> husband2;
husband2.reset(w1);             // husband2 now manages w1

cout << *husband << endl;            // Segmentation fault
```
Note that when **husband2** takes over **w1**, it firstly takes away the ownership to **w1** of **husband**. So **husband** only has a **nullptr** afterwards.

For the sake of multiple smart pointers pointing to the same object, C++11 introduces **shared_ptr**.
***
### shared_ptr
**shared_ptr** keeps tract of the number of referencing to the object, which can be gained by member function use_count();
```cpp
shared_ptr<Person> sp1;

shared_ptr<Person> sp2(new Person(2));

cout << "sp1	use_count() = " << sp1.use_count() << endl;         // 0
cout << "sp2	use_count() = " << sp2.use_count() << endl;         // 1

// sharing
sp1 = sp2;

cout << "sp1	use_count() = " << sp1.use_count() << endl;         // 2
cout << "sp2	use_count() = " << sp2.use_count() << endl << endl; // 2

shared_ptr<Person> sp3(sp1);
cout << "sp1	use_count() = " << sp1.use_count() << endl;         // 3
cout << "sp2	use_count() = " << sp2.use_count() << endl;         // 3
cout << "sp2	use_count() = " << sp3.use_count() << endl << endl; // 3
```
when the last pointer to the object is dead, i.e. use_count() returns 0, the memory that this **shared_ptr** manages is freed.
#### Construction
1). **shared_ptr< T > sp1**:  a null **shared_ptr**

```
shared_ptr<Person> sp1;
Person *person1 = new Person(1);
sp1.reset(person1);
```
2). **shared_ptr< T > sp2(new T())**: 
```cpp
shared_ptr<Person> sp2(new Person(2));
shared_ptr<Person> sp3(sp1);
```

3). **shared_ptr<T[]> sp4**: a null **shared_ptr**，which can point to array of type T, supported after C++17
```cpp
shared_ptr<Person[]> sp4;
```

4). **shared_ptr<T[]> sp5(new T[] { … })**:  point to array of type T, supported after C++17
```cpp
shared_ptr<Person[]> sp5(new Person[5] { 3, 4, 5, 6, 7 });
```

5). **shared_ptr< T > sp6(NULL, D())**: a null **shared_ptr**，accept a function(can also use a lambda expression) as the deleter to free the memory
```cpp
shared_ptr<Person> sp6(NULL, DestructPerson());
```
6). **shared_ptr< T > sp7(new T(), D())**: a **shared_ptr** pointing to type T，accept a function(usually a lambda expression) as the deleter to free the memory
```cpp
shared_ptr<Person> sp7(new Person(8), [](){
    /*deleting...*/
});
```
7). **make_shared** function: allocates a piece of memory for the object and return the pointer to it (**recommended** for better efficiency in memory management)
```cpp
shared_ptr<int> up3 = make_shared<int>(2);
```

#### To be noted
1) **shared_ptr** inside a **container** should be **erased** when it is no longer used, since it will **NOT** release the memory automatically like a normal **shared_ptr**.
```cpp
list<shared_ptr<string>>pstrList;
    pstrList.push_back(make_shared<string>("1111"));
    pstrList.push_back(make_shared<string>("2222"));
    pstrList.push_back(make_shared<string>("3333"));
    pstrList.push_back(make_shared<string>("4444"));
    for(auto p:pstrList)
    {
        if(*p == "3333");
        {
            /*do some thing!*/
        }
        cout<<*p<<endl;
    }
    /*finish using 3333 data！*/
    

    for(list<shared_ptr<string>>::iterator itr = pstrList.begin();itr!=pstrList.end();++itr)
    {
        if(**itr == "3333"){
            cout<<**itr<<endl;
            pstrList.erase(itr);    // erase the useless shared_ptr
        }
    }
```
2) A self-defined **deleter** must be provided for **Dynamic array** memory management of **shared_ptr**
```cpp
shared_ptr<DelTest> p(new DelTest[10],[](DelTest *p){delete[] p;});
/* use lambda function as a self-defined deleter */
```
3) **Never, ever** initialize multiple **shared_ptr** using the same pointer
4) Cannot use a **naked pointer** for initializing
```cpp
// shared_ptr constructor is explicit type，cannot do this
std::shared_ptr<int> p1 = new int(); // can't implicitly convert, type mismatch
int* p11 = new int;
std::shared_ptr<int> p12(p11);  // no problem
```
5) **DO NOT** use a pointer in stack (instead of **heap**) for initializing
```cpp
int x = 12;
std::shared_ptr<int> ptr(&x);   // no way bro
```
6) **DO NOT** use the return value of get() of a **shared_ptr** to initialize another **shared_ptr**
```cpp
Base *a = new Base();
std::shared_ptr<Base> p1(a);
std::shared_ptr<Base> p2(p1.get());
/* p1 and p2 have different number of reference to the memory */
/* this may lead to duplicated deletion to the same memory, which is dangerous */
```
7) **Multithreading**
   
**use_count()** method is implemented with lock and maintains thread security. However a **shared_ptr** can be used in multiple threads for IO(like construction and destruction), which may not be **atomic**.
A **lock** is required for multithreading with **shared_ptr**.

8) Avoid **corss-use**
If object A has a **shared_ptr** member pointing to object B, whereas object B also has a **shared_ptr** member pointing to object A, the memory that A and B occupies will **NEVER** be released.(Not hard to derive this conclusion).

But in reality, from time to time we need cross-reference. Hence **weak_ptr** is introduced.
***
### weak_ptr
**weak_ptr** is introduced for collaborating with **shared_ptr**. 
1) It does not overload * and -> operator, which means it cannot mutate the memory it points to. It can use **lock()** to get a **shared_ptr** to the memory
2) It won't increase or decrease the number of reference recorded by a **shared_ptr**
3) It acts as the observer of the resource held by a **shared_ptr**. It can use **expired()** to check whether **use_count()** returns 0 (but much faster)
#### Construction
```cpp
weak_ptr<int> wp1;                  // NULL
weak_ptr<int> wp2(wp1);             // copy construction
weak_ptr<int> wp2 = wp1;            // copy construction
  
shared_ptr<int> sp(new int);
weak_ptr<int> wp3(sp);              // can construct from a shared_ptr
weak_ptr<int> wp3 = sp;
  
shared_ptr<int> sp1 = wp3.lock()    //return a shared_ptr
if (wp3.expired()){
    /* expired() returns true when the memory is released */
    /* lock() return nullptr in this case */
    cout << "Expired!\n";
}   
```
#### Thread security for shared objects
In multithreading scenario, it is possible that when a object is being destructed, its method is used by other threads. For example:
```cpp
class Test {
  public:
    Test(int id) : m_id(id) {}
    void showID() {
      std::cout << m_id << std::endl;
    }
  private:
    int m_id;
};
  
void thread1(Test* t) {
  std::this_thread::sleep_for(std::chrono::seconds(2));
  t->showID();  // object's method is used when it is destructing
}
  
int main()
{
  Test* t = new Test(10);
  std::thread t1(thread1, t);
  delete t;  // t is destructed here
  std::cout << "delete t" << std::endl;
  t1.join();

  return 0;
}
```
Thread t sleeps for 2 seconds before calling the object method, at that time the object is dead already. To prevent this, let's take a look at the below implementation with **shared_ptr** and **weak_ptr**:
```cpp
class Test {
  public:
    Test(int id) : m_id(id) {}
    ~Test() {std::cout << "~Test" << std::endl;}
    void showID() {
      std::cout << "m_id: " << m_id << std::endl;
    }
  private:
    int m_id;
};
  
void thread2(std::weak_ptr<Test> t) {
  std::this_thread::sleep_for(std::chrono::seconds(1));
  std::shared_ptr<Test> sp = t.lock();  // return NULL when the memory has been released
  if(sp)
    sp->showID(); // print：2
  else
    std::cout << "sp null" << std::endl;
}
  
int main()
{
    std::thread t2;
    // {
        std::shared_ptr<Test> sp = std::make_shared<Test>(2);
        std::weak_ptr<Test> t(sp);
        t2 = std::thread(thread2, t);
        std::cout << "use_count: " << sp.use_count() << std::endl;
    // }
    t2.join();

  return 0;
}
```

### Conclusion
**unique_ptr** : monogamy

**shared_ptr** : Polygamy

**weak_ptr**   : can't live without Polygamy