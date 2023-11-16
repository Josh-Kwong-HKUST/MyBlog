# inline in C++
###### Latest edited: Sep 20, 2023
***
### What is inline function?
Similar to **define**, **inline** is a keyword to replace the function invoke with the function body during compilation time. For example, in the below C++ code, each invoke of function **check** wil be replaced by **(i % 2) ? "odd" : "even"**.
```cpp
inline const char* check(int num){
    return (num % 2) ? "odd" : "even";
}

int main(){
    for (int i = 0; i < 100; i++){
        std::cout << check(i) << std::endl;  // to be replaced
    }
    return 0;
}
```
### Why inline function?
Well, **inline** is used to reduce the cost of invoking **small function**, therefore improve efficiency but larger executable file size(**space for time**). However, it is not guaranteed that the function will be inlined. The compiler will decide whether to inline the function or not. For example, the compiler will not inline the function if the function is recursive or the function is too large.

### When & How to use inline function?
- **When**: 
    - The function is small and simple.
    - The function is called frequently.
    - The function is not recursive, not including complex control statement like **while**, **switch**, etc.
    - The function is not virtual.
- **How**:
    - Define the inline function in the header file.(declaration and definition in the same file)
    - inline only works when put before the function definition.

### A few more words...
- **inline** is just a suggestion to the compiler, not a command.
- Multi-threading using **inline** won't affect performance.
- We can use **\_\_attribute\_\_((noinline))** for gcc or **\_\_declspec(noinline)** for MSVC to prevent the compiler from inlining the function.
