## lambda expression
**lambda expression** is also called **anonymous expression**. It is usually a function without a name. It is mostly used as a parameter to another function. It is a very powerful feature introduced in C++11, a great tool to write concise and readable code.
```
[](string name)
{
   cout << "this is anonymous" << endl;
   cout << "hello " << name << endl;
}("josh");
```
Above is a simple sample of a lambda expression. [] is used to capture variables from the outside scope, which will be discussed in detail later. () is used to pass parameters to the lambda function. {} is used to define the body of the lambda function. The last line is to call the lambda function with parameter "josh".
We can use **auto** to store a lambda expression as below.
```
auto func = [](string name)
{
   cout << "this is anonymous" << endl;
   cout << "hello " << name << endl;
};
func("josh");
```
or we can use a function pointer to store a lambda expression.
```
typedef void (*funcPtr)(string);
funcPtr func  = [](string name)
{
   cout << "this is from funcPtr" << endl;
   cout << "hello " << name << endl;
};
func("josh");
```
Moreover, we can use **std::function**(new feature in C++11) to store a lambda expression.
```
std::function<void(string)> func = [](string name)
{
   cout << "this is from std::function" << endl;
   cout << "hello " << name << endl;
};
func("josh");
```
### **Capture**
As mentioned above, [] is used to capture variables from the outside scope. There are three ways to capture variables: **by value**, **by reference** and **by both**.
```
int age = 33;
string name = "josh";
int score = 100;
string job = "softengineer";
// capture by value
[age, name](string name_)
{
    cout << "age is " << age << " name is " << name << " self-name is " << name_ << endl;
}("Gordon");
```
**age** and **name** are captured **in value** above, and they cannot be modified inside the lambda function. If you try to modify them, compiler will throw an error warning that you are trying to mutate a **const** variable.