# Cover page
## Mesaure performance in a for loop
Some developer may bench the performance by following code
```c
int main(){
    int startTime1 = getTime();
    for(int i = 0; i<10000000;i++){

    }
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    int startTime2 = getTime();
    for(int i = 0; i<10000000;++i){
    
    }
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
}
```
Question: Which one is correct
A: I did similar bench in the past
B: No, I didn't
Answer:
compile can optimize out empty blocks including empty if/for/while etc.So the code become
```c++
int main(){
    int startTime1 = getTime();
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    int startTime2 = getTime();
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
}
```
So the final timeCost1 and timeCost2 and very small numbers. And the actual value depends on the OS interference factors, such as scheduling and timer interrupt


## i++ and ++i
```c
for(int i = 0; i<10000;i++){
    //some code
}
```

```c
for(int i = 0; i<10000;++i){
    //some code
}
```
Question: which is faster
Options:
A: i++ faster
B: ++i faster
C: same

Answer the performance is the same because compiler eliminate unused value assignment.
```c
tmp = i
i = i + 1
```

```c
i = i + 1
tmp = i
```
## swap two variables
```c
int temp = a;
a = b;
b = temp;
```

```c
a = a ^ b;
b = a ^ b;
a = a ^ b;
```
Question: Which way should we use to swap two variables? Is avoiding `temp` faster?

Answer: On modern processors, a normal variable swap can be handled efficiently with registers and register renaming. XOR swapping occupies the ALU with several dependent instructions, so instruction-level parallelism is poor.

## how to iterate over an array
```c++
for(int i = 0;i<array.size();i++){

}
```

```c++
int size = array.size();
for(int i = 0;i<size;i++){
    
}
```
Question: which one is faster? why?
Answer:
Compiler has loop invariant code motion. The array.size() function will be called multi times during the loop when the array size is unchanged in the loop. Moreover, compiler also inline small function calls, so the `.size()` function call will not exist. Instead, it will be optimized to `array._size` attribute read.
`int size = array.size();` is also not a performance problem, because compiler has variable liveness analysis. The storage allocated for `int size` can be reused by other variables after the for loop.

## named constants vs magic number
```c
int hoursInDay = 24;
int minutesInHour = 60;
int secondsInMinute = 60;

int secondsInDay = hoursInDay * minutesInHour * secondsInMinute;
```

```c
int secondsInDay = 86400;
```
Question: Which version is easier to understand? Does the first version do more work at runtime?

Answer: The named constants make the units and intent clear. The compiler has const propagation to calculate the product ahead of time, so the first version does not need to perform those multiplications at runtime. Prefer the readable version over the magic number. One more note, some programming language can declare variable as const, but compiler const propagation doesn't rely on the const qualifier.

## variable reuse
```c
int milage = getMilage();
dataProcess1(milage);
int velocity = getVelocity();
dataProcess2(velocity);
```

```c
int val = getMilage();
dataProcess1(val);
val = getVelocity();
dataProcess2(val);
```
Question: Some developers changed code snap 1 to code snap 2, what did they aim to do? Does it work?

Answer: compiler's liveness analysis can reuse storage of variables, so developers don't need to reuse variable to save memory.
Similarly, there is no need to remove meaningful intermediate variable. 
```c
int priceAfterDiscount = price - discount;
int finalPrice = priceAfterDiscount + shipping;
```

## x/2 vs x>>1
```c
int foo(int diameter){
    int radius = diameter/2；
    return radius;
}
```

```c
int foo(int diameter){
    int radius = diameter>>1；
    return radius;
}
```
Question:
Which one do you prefer? Why?

Answer:
Compiler has strength reduction, `x/2` will be optimized by `x>>1` automatic, so developers can leave `x/2` to keep code readable.

## how to bench x/2 and x>>1
```c
int main(){
    int startTime1 = getTime();
    for(int i = 0; i<10000000;i++){
        int x = i/2;
    }
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    int startTime2 = getTime();
    for(int i = 0; i<10000000;++i){
        int x = i>>1;
    }
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
}
```
Question: what's the result of this bench
Answer: As mentioned above
1. compiler can optimize out unused value assignment
2. compiler can optimize out empty `for` blocks
3. compiler run optimization passes may rounds, and each pass have dedicated target.
```c++
int main(){
    int startTime1 = getTime();
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    int startTime2 = getTime();
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
}
```
## how to bench x/2 and x>>1 further
```c++
int main(){
    int startTime1 = getTime();
    int sum1 = 0;
    for(int i = 0; i<100;i++){
        int x = i/2;
        sum1 += x;
    }
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    print(sum1);
    int startTime2 = getTime();
    int sum2 = 0;
    for(int i = 0; i<100;++i){
        int x = i>>1;
        sum2 += x;
    }
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
    print(sum2);
}
```
Question: Can this code get correct bench result? Why?
Answer:
Compiler has loop variable induction‌ and const propagation. The code will be optimized to print a compile time const.
```c++
int main(){
    int startTime1 = getTime();
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    print(2450);
    int startTime2 = getTime();
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
    print(2450);
}
```
# What compiler can't do
## code rely on runtime information
```c
int random();

int main(){
    int startTime1 = getTime();
    int sum1 = 0;
    for(int i = 0; i<100;i++){
        int x = random()/2;
        sum1 += x;
    }
    int endTime1 = getTime();
    int timeCost1 = endTime1-startTime1
    print(timeCost1);
    print(sum1);
    int startTime2 = getTime();
    int sum2 = 0;
    for(int i = 0; i<100;++i){
        int x = random()>>1;
        sum2 += x;
    }
    int endTime2 = getTime();
    int timeCost2 = endTime2-startTime2
    print(timeCost2);
    print(sum2);
}
```
Question: Why compiler can't optimize this code?
Answer:
If code rely on runtime information, then compiler doesn't have enough context to optimize it at compile time.
## wrong data structure selection
```c++
array<int> container;
void producer(int value){
    container.push(value);
}

void consumer(){
    for(value of container){
        //some code
    }
}
```

```c++
set<int> container;
void producer(int value){
    container.push(value);
}

void consumer(){
    for(value of container){
        //some code
    }
}
```

```c++
list<int> container;
void producer(int value){
    container.push(value);
}

void consumer(){
    for(value of container){
        //some code
    }
}
```
Question: Which has better performance? Why?
Answer: The data is only pushed and iterated in a linear way and array push is O(1) and array is in a contiguous memory which is more cache friendly. set is usually implemented by hash map or RB-Tree. Hash map has hash value overhead and RB-Tree is O(logn). Moreover, hash map, RB-Tree and linked list are not memory contiguous as array.
And most compilers doesn't have modeling and analysis of data structures. So if the data structures are chosen wrong, the code remains slow.
## code with side effect
1
```c++
log(...)
```
2
```c++
int x;
void foo(){
    x = 10;
}
```
3
```c++
void foo(int* p){
    p = 5;
}
```

4
```c++
void foo(){
atomic<int> x;
x = 7;
}
```
Question:

Map following side effect to above code:
Options:
A: value assignment to global variable
B: system IO
C: multi thread atomic variable access
D: memory alias analysis is a hard challenge for compiler
Answer:
All options are often seen side effect in code.
On production, developers may be required to reduce log for better performance, this work can't be done by compiler, because compiler won't optimize out IO side effect.
