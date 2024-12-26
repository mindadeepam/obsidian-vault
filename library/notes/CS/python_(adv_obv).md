#python

## decorators
[link](https://www.geeksforgeeks.org/decorators-in-python/)
### code for decorator chaining 

```python
def decor1(func): 
    def inner(): 
        x = func() 
        return x * x 
    return inner 

def decor(func): 
    def inner(): 
        x = func() 
        return 2 * x 
    return inner 

@decor1
@decor
def num(): 
    return 10

@decor
@decor1
def num2():
    return 10
  
print(num()) 
print(num2())

Output:

400
200
```
The above example is similar to calling the function as –

```python
decor1(decor(num))
decor(decor1(num2))
```

Decorators are used to modify the behavior of functions or methods. Use them when you want to add functionality like logging, caching, or authentication to existing functions without modifying their source code. They help in separating concerns and improving code readability.



## @staticmethod

When function decorated with @staticmethod is called, we don’t pass an instance of the class to it as it is normally done with methods. It means that the function is put inside the class but it cannot access the instance of that class.

- only used as utlitlyi functions defined inside a class just for convinience.

Example #1:

```
# Python program to  
# demonstrate static methods 
  
class Maths(): 
      
    @staticmethod
    def addNum(num1, num2): 
        return num1 + num2 
          
# Driver's code 
if __name__ == "__main__": 
      
    # Calling method of class 
    # without creating instance 
    res = Maths.addNum(1, 2) 
    print("The result is", res) 
```

## @classmethod

•	This method gets the class itself as its first argument (usually named cls).
•	It can modify class-level data or create instances of the class.

 ***Factory method using a Class Method***
A common use case for class methods is defining factory methods. Factory methods are methods that return an instance of the class, often using different input parameters. ie `AutoModel.from_pretrained` much?

***What is the difference between a method and a class method?***
- ****Method****:
    - ****Instance Method****: Takes `self` as the first parameter. It is used to access or modify instance attributes and can call other instance methods.
    - ****Example****:
        
        class MyClass:  
            def instance_method(self):  
                return "This is an instance method"  
        
- ****Class Method****:
    - ****Class Method****: Takes `cls` as the first parameter. It is used to access or modify class state that applies across all instances.
    - ****Decorated with****: `@classmethod`
    - ****Example****:
        
        class MyClass:  
            @classmethod  
            def class_method(cls):  
                return "This is a class method"

## #websocket vs #http

- harkirat singh websockets explained: https://www.youtube.com/watch?v=7WQ2EbXLfL.I see [[websockets]]
- google webrtc presentation: https://www.youtube.com/watch?v=p2HzZkd2A40 

## async:
https://chatgpt.com/share/66e3714a-5588-800c-ae22-fee4f2536c5b