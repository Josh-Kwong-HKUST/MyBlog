## Type Casting in C++

### Implicit Casting
When a value is assigned to a compatible type, there is **implicit casting** behind the scene done by the compiler.
```cpp
short a = 2000; // cast int to short
int b = a;      // cast short to int
```

For non-primitive type, array and function, they will be casted to pointer implicitly. Meanwhile pointers are allowed as follow:
1) Null pointer can be casted to any pointer types
2) Any types of pointers can be casted to void*
3) Upcasting: a pointer to **derived class** can be casted to a pointer to **base class** without modifying its **const** or **volatile** properties
   
A common principle of implicit casting is **from low precision to high precision**
```cpp
int a = 3;
double b = 4.5;
a + b;  // a is casted to double implicitly before addition operation
```
```
// assigning value to different types
int a = true;       // bool to int
int* ptr = nullptr; // nullptr to int*
```
```cpp
void func(double a);
func(1);    // 1 is implicitly casted to double: 1.0
```
```cpp
double add(int a, int b){
    return a + b;   // the result will be casted to double before return
}
```
Implicit conversions also include constructor or operator conversions, which affect classes that include specific constructors or operator functions to perform conversions. For example:
```cpp
class A {};
class B { public: B (A a) {} };
  
A a;
B b=a;
```
#### Risk of implicit casting
By default, a constructor with only one parameter defines a implicit casting, which can be problematic.
```cpp
class Test{
public:
    Test(int a):val(a){}
    bool isSame(Test other){
        return val == other.val;
    }
private:
    int val;
}
  
int main(){
    Test a(10);
    std::cout << a.isSame(10) << std::endl; // 10 is casted to Test instance! return true!
    return 0;
}
```

#### Keyword: explicit
We can add **explicit** keyword to the declaration of the constructor to prohibit implicit casting
```cpp
class Test{
public:
    explicit Test(int a):val(a){}
    bool isSame(Test other){
        return val == other.val;
    }
private:
    int val;
}
  
int main(){
    Test a(10); // not gonna compiled!
    std::cout << a.isSame(10) << std::endl;
    return 0;
}
```
***
### Explicit Casting
C++ is a **strongly typed** programming language. Hence we need **explicit casting** from time to time. There are two generic ways to do so:
```cpp
double x = 10.1;
int y;
y = int (x);    // functional notation
y = (int) x;    // c-like cast notation
```
They are good enough for the most scenarios. But they can also be problematic sometimes.
```cpp
class foo{
    void g();
}
class bar{}

int main(){
    // case 1:
    foo f;
    bar* ptr = (bar*)&d;    // allowed
    ptr->g();               // There is no g() in foo class! but complier does no complain.

    // case 2:
    int a = 5;
    const int* p1 = &5;
    *p1 = 6;                // error, p1 points to const int*
    int *p2 = (int*) p1;
    *p2 = 6;                // allowed!
    std::cout << a << std::endl;// a is 6 now
}
```
There are scenarios where complier does not complain but runtime errors occur, which is dangerous.
C++ introduces 4 types of **explicit casting**
#### static_cast<T>
**static_cast** is usually used for casting **non-polymorphic type**. It is similar to C-like casting, but note that it cannot reduce the **const**, **volatile** or **__unaligned** attribute of the original type.
```cpp
char a = 'a';
int b = static_cast<char>(a);   // char to int

double *c = new double;
void *d = static_cast<void*>(c);// double* to void*

int e = 10;
const int f = static_cast<const int>(e);// int to const int

const int g = 20;
int *h = static_cast<int*>(&g);// compilation error: cannot reduce the const attribute
```
#### dynamic_cast
**dynamic_cast** casts pointers to class, reference or void*. It ensures that the casting result points to a complete and valid object with the destination type (this differentiates it from **static_cast**)

Upcasting:  cast derived class pointer to base class pointer, same way as implicit casting
Downcasting: cast base class pointer to derived class pointer. The base class should be polymorphic (class with virtual method)

Return:
1)  If **failed** to cast a pointer, return **nullptr**
2)  If **failed** to cast a reference, throuw **std::bad_cast** exception(defined in **<typeinfo>**)

```cpp
class Base{
    virtual void func(){ cout << "Base!" << endl; }
}
class Derive: public Base{
    void func() override { cout << "Derive!" << endl; }
}
int main(){
    // Upcasting
    Derive d1 = new Derive();
    cout << "d1: " << d1 << endl;
  
    Base* b1 = dynamic_cast<Base*>(d1);
    cout << "b1: " << b1 << endl;
  
    // Downcasting
    Base* b2 = new Base();
    cout << "b2: " << b2 << endl;
  
    Derive* d2 = dynamic_cast<Derive*>(b2);
    cout << "d2: " << d2 << endl;
  
    return 0;
}
```
Output:
```bash
d1: 0x7fdf98c057a0
b1: 0x7fdf98c057a0
b2: 0x7fdf98c057b0
d2: 0x0
```
d2 is set to nullptr after downcasting because it is what dynamic_cast does to ensure safety

#### const_cast
**const_cast** can increase or reduce the mutability (**const**) or volatility (**volatile**) of a pointer.
```cpp
void print(char* str){
    cout << std << '\n';
}

int main(){
    const char* c = "sample";
    print(const_cast<char*>(c));
    return 0;
}
```
The above example does not modify the data that the pointer points to, which is totally fine. However it is dangerous (and undefined behaviour) to write to the data with removing its **const** attribute.

#### reinterpret_cast
This is the most **unsafe** casting, which is not recommended to use. It does no checking before the casting, which means you can cast anything (yes, any types you can come up with) to a pointer pointing to **any types**.
```
int main()
{
    int* p = new int(5);
    uint64_t p_val = reinterpret_cast<uint64_t>(p);

    cout << "p    :" << p << endl;
    cout << "p_val:" << hex << p_val << endl;

    return 0;
}
```
The code convert the address which p points to, to a **uint64_t** integer, and you can do it the other way around.
***
### Summary
**static_cast**     : primitive type casting, low risk

**dynamic_cast**    : up and down casting for polymorphic classes, low risk

**const_cast**      : remove or add **const** attribute to the pointer, low risk (unless you overwrite after removing it)

**reinterpret_cast**: freestyle, high risk, **DON'T USE**