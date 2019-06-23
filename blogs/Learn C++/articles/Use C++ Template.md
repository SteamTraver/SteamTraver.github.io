# 关于C++模板的使用

## 问题1：用模板函数或者宏返回类型为T的数组a[0~n-1]的元素个数

### Here are the methods I've tried

M0: Try to use for-loop under condition of `a[i] != null`. And use variable `i` to store the elements' number. Code Below:  

```c++
template<typename T>
int count(T a[])
{
    int i = 0;
    for(;a[i] != null;)
        ++i;
    return i;
}
```

Obviously, it doesn't work. The Condition `a[i] != null` is totally wrong:  

1. Value of an item which is out of this array, cannot be ***null***. If T is Int, then `a[n]` is an int value, same as a char when T is Char.

2. This operation might cause memory problem, not just for I accessed the `a[n]` that haven't been define, but also may cause an excpetion when in Java Environment.

M1: Try use a pointer to determine the boundary of this array. Code much like the upper one, and it failed.  

***So, let's make a conclusion that it seems impossiable to determine an array's size by enumerating it's elements in a loop when you just got an array as a arg in a function. So I gaved up trying enumration***

M2: Then I found two methods:  

1:

```c++
template<typename T, size_t N>
size_t array_size(T (&) [N])
{
    return N;
}
```

2:

```c++
template<typename T>
size_t countof(T (&a))
{
    return sizeof(a)/sizeof(a[0]);
}
```

**And They Actually Worked!(While 1 cannot accept an array with 0 size)** Test below:  

```c++
int main()
{
    //测试一个可以获取任意类型数组的元素的模板函数
    int a[5] = {1,2,3,4,5};
    char b[12]="hello,world";
    int c[0];

    size_t size_of_array1 = countof(a);
    size_t size_of_array2 = countof(b);
    size_t size_of_array3 = countof(c);

    size_t size_of_array4 = array_size(a);
    size_t size_of_array5 = array_size(b);

    //size_t size_of_array6 = array_size(c);
    cout<<"countof output: \n";
    cout<<size_of_array1<<"\t"<<size_of_array2<<"\t"<<size_of_array3<<"\n";
    cout<<"array_size output: \n";
    cout<<size_of_array4<<"\t"<<size_of_array5;

    return 0;

}
```

and the ![output in vscode under MingW64]:<https://github.com/SteamTraver/SteamTraver.github.io/blob/dev/blogs/Learn%20C%2B%2B/images/2.jpg>  

***There comes some questions about these functions:***  

1. Why they use ***reference*** instead of ***pointer***?  
Check this difference: `array_size(T (&) [N])`between `array_size(T a[N])`. When use **pointer**, the size lost, which won't happen when **refernece** is being used. You can basicly take that `T (&) [N]` represents the **whole physical memory of array**, when you pass it to the function, you actually got the whole array,and it has the real size of the array that has been calculated by systeam. If you use the pointer, you cannot get the actual size, cause **pointer** just only represents the address of first element in the array. And you should be careful that you should add `()` to wrap `&` since the priority of operator combination.  

2. Why can ***1*** get the size of array directly?  
When you passed the arg to the function, the system will compute the size. All you need to do is to return it.

3. Do they have shortcomings?  
Obviously, they do. Both of them are restricted with array arg, not non-array arg. They cannot deal  with ***pointer*** and ***class object with inner array***.

### Make them better

We already got the solutions, but can we improve it? When we want to run these functions constantly or need to create a new array whose size is based on the former one? We can turn these functions into `#define` block.  

1:  

```c++
#define countof(array) (sizeof(array)/sizeof(array[0]))
```

>notes: `countof` cannot deal well with pointer or object as arg.  

2:  

```c++
template<typename T,size_t N>
char(& ArraySize(T (&)[N]))[N];

#define count(array)(sizeof(ArraySize(array)))

```

note: `ArraySize()` is a function that returns ***reference*** to a char array with N elements. And `ArraySize()` accepts the ***reference to T array of N elements*** as argment. `ArraySize()` just need declaration, not definition.  

I got little confused about the work of `ArraySize()`. Why it does't need a definition?  
I tend to think that `ArraySize()` get the whole memory of array because that reference arg. And `char(& )[N]` get it too, and store it in a char[N].  

### Ending

So this is something that I've found about this simple question. By the way this is my first Blog, I decided to post it on my [github account](https://github.com/steamtraver).  
(2019/6/23)  
