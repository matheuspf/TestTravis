# Handy

[![Build Status](https://travis-ci.org/matheuspf/TestTravis.svg?branch=master)](https://travis-ci.org/matheuspf/TestTravis) [![Coverage Status](https://coveralls.io/repos/github/matheuspf/TestTravis/badge.svg?branch=master)](https://coveralls.io/github/matheuspf/TestTravis?branch=master)

Handy is a compilation of very useful header only utilities for modern C++.

No need to compile or install anything, simply include and use.

All you need is a compiler supporting C++11/14/17.

There are many tests and examples




<br>

- [Handy](#handy)
    - [Algorithms](#algorithms)
    - [Container](#container)
    - [Helpers](#helpers)
        - [Benchmark](#benchmark)
        - [HandyParams](#handyparams)
        - [HasMember](#hasmember)
        - [NamedTuple](#namedtuple)
        - [Print](#print)
        - [Random](#random)



<br><br>


## Algorithms

Define STL container algorithms and pipeline operators.

By including this class, for almost any stl algorithm, a function taking a container instead of iterators is defined.
             
This function simply delegate the call to the actual stl function by calling std::begin and std::end on the container passed as parameter.
            
The exact form of the function, as well as the number of arguments depends on the stl algorithm. For each begin/end pair there is one container. For single iterators, such as the third argument to std::copy, there is also one container. Any other paramater of the stl algorithm can also be passed as parameter to the handy version.

An example of use:

``` C++

std::vector<int> v, u;

handy::sort(v, std::greater<int>());

handy::copy(v, u);

std::vector<int> transformed = handy::transform(v, u, [](int x){ return 2 * x; });
```

Notice that the return is also modified for some functions. For stl functions that do not return, a reference to the container that is being modified is returned. 

For functions that return an iterator that does not imply position (std::transform, for example), the return is also the container that is being modified. Any other case has the same return as the stl algorithm.

Another feature is the function pipeline, which can be controled by changing the **ALGORITHM\_OP\_SYMBOL** macro (the default is **&**).
             
By passing only the arguments to the handy algorithm function, a lambda is returned, taking the missing container argument.

It is easy then to construct a pipeline of operations (starting at the left). Also, the order of the parameters is irrelevant, as long as a single container is given. Example:

``` C++
std::vector<int> sortedLeft = v & handy::sort();

std::vector<int> sortedRight = handy::sort() & v;

int sum = u & handy::reverse() & handy::accumulate(0);

auto finder = handy::find_if([](auto x){ return x == 10; });

auto posV = (v & finder) - v.begin();

bool posU = finder(u) - u.begin();
```

For many functions the overloads are not ambiguous to the original stl functions, so there is no need to use the handy namespace to disambiguate the call.

<br><br>

## Container

An interface to easily access multidimensional data, having total compatibility with STL algorithms and containers

it is a very generic and easy to use multidimensional container, which allows easy creation and access.

It is aimed to be fast and easy to use.

Also, there is a lot of ways to use it:


``` C++
// Type int and three dimensions with sizes 10, 20 and 30 respetivelly.
// In this case (compile time size is given), 'a' inherits from 'std::array'.
handy::Container<int, 10, 20, 30> a;

/// The same as above, but now 'b' inherits from 'std::vector' (runtime size is given).
handy::Container<int> b(10, 20, 30);

// You can also create a Container giving the sizes via an itearable type
// (any type that has both std::begin and std::end defined). They have
// to contain any integral type.
handy::Container<double> c(vector<int>{10, 20, 30});

// Or using more than one iterable (different types, but all of them are integral)
handy::Container<char> d(list<char>{10}, set<long>{20}, array<int, 1>{30});

// You can also use a range defined by iterators (or pointers)
int szs[] = {10, 20, 30, 10000000};

handy::Container<long> e(&szs[0], szs + 3);

// Or even a initializer list of integrals
handy::Container<float> f({10, 20, 30});


// These are some ways to access the containers via 'operator()'

// With variadic integrals
a(0, 1, 2) = 10;

// With iterables
b(vector<int>{3}, list<char>{4, 5}) = 20;

// With integrals or iterables (only for access, not allowed for creation)
c(9, set<int>{6, 7}) = 0.1;

// With list initialization
d({9, 19, 29}) = 'a';

// With a tuple of integrals/iterables (also only for access)
e(make_tuple(9, 19, 29)) = 20;

// And you can access it normally with 'operator[]'
f[50] = -10.0;

// You can also use any STL function. It is either a std::vector or a std::array, after all
sort(a.begin(), a.end());
```

You can also take a **slice** of a container like this:


``` C++
handy::Container<int> cnt(2, 3, 4);

std::iota(cnt.begin(), cnt.end(), 0);   // [0, 1, ..., 23]

auto slice1 = cnt.slice(1);  // [12, 13, ..., 23]

auto slice2 = cnt.slice(1, 2); // [20, 21, 22, 23]

auto slice3 = cnt.slice(0);  // [0, 1, ..., 11]

auto slice4 = cnt.slice(0, 1);  // [4, 5, 6, 7]

auto slice5 = cnt.slice(1, 2, 3);   // [23]

// The access is exactly the same as in the Container, but now in the new dimensions
auto x = slice1(0, 1);
```

<br><br>

## Range





<br><br>



## Helpers



### Benchmark

Precise time measurement for both Windows and Linux (up to nanoseconds). The time return is in **seconds**.

I did this one because the standard library for C++ kinda lacks precision. Also, this one is much easier to use:

``` C++
    handy::Benchmark bm;

    // Do something

    double timeElapsed = bm.finish();

    bm.start();

    // Do something else

    double moreTimeElapsed = bm.finish();  // Time until last call to start()
```

Also works with a function call:


``` C++
    handy::Benchmark bm;

    auto func = []{ /* Do something */ };

    double timeElapsed = bm(func);

    double moreTimeElapsed = handy::benchmark(func); // Same as above
```

<br>


### HandyParams

This is a very simple header that allows the creation of classes with (almost) python-like initialization

It is only supported in C++17, with the aid of the [std::any](http://en.cppreference.com/w/cpp/utility/any) class. Also, it incurs a small runtime overhead, and may be prone to type casting rrors, but the result is simply beautiful:


``` C++
struct Foo
{
    int a;
    double b;
    float c;
    std::string d;


    // Put this at a public section of your class with the class
    // name and all variables you want to be able to initialize

    HANDY_PARAMS(Foo, a, b, c, d);
};


int main ()
{
    // A std::map<std::string, std::any> is passed in the constructor.
    // Notice that you just need to inialize the variables you want.
    // Also, the type must be exact (10.0f instead of 10.0 for floats)

    Foo foo({{"a", 3}, {"c", 10.0f}, {"d", "abcdefg"s}});

    return 0;
}
```

<br>


### HasMember

Simple utility to verify at compile time if a class has a variable or a function. 

``` C++
struct Foo
{
    void foo (int) {}

    void bar (int) {}
    void bar (std::string) {}

    int var;
};

struct Goo {};


void foo (int) {}

void foo (Foo, std::string, Goo = Goo{}) {}


HAS_VAR(var);   // Check the existence of the variable 'var'. Automatically named 'Hasvar'

HAS_FUNC(foo, HasFoo);   // Check the existence of the regular member function 'foo'. Manually named 'Hasfoo'

HAS_EXTERN_FUNC(foo, HasExternFoo);     // Check the existence of the free function 'foo'

HAS_OVERLOADED_FUNC(bar, HasOverloadedBar);     // Check the existence of the overloaded member function 'bar'


int main ()
{
    Hasvar<int>::value;     // false
    Hasvar<Foo>::value;     // true

    HasFoo<Foo>::value;     // true
    HasFoo<Goo>::value;     // false

    HasExternFoo<Foo>::value;   // false
    HasExternFoo<Foo, std::string>::value;   // true
    HasExternFoo<Foo, std::string, Goo>::value;   // true

    HasOverloadedBar<Foo, std::string>::value;   // true
    HasOverloadedBar<Foo, double>::value;   // true -> implicit call to bar(int)

    return 0;
}
```

<br>




### NamedTuple

Very simple macros for providing named access to std::tuple
    
You can create a class that inherits from std::tuple, having access functions defined by you. There is no space (for each object) or runtime overhead. Ex:

``` C++
// Define a struct called Triplet having functions first, second and third
NAMED_TUPLE(Triplet, first, second, third)

int main ()
{
    Triplet<int, double, std::string> triplet;

    std::get<0>(triplet) = 10;  
    
    triplet.first() = 10;       // Exactly the same as above
    triplet.second() = 20.0;
    triplet.third() = "abcdef";

    return 0;
}
```

Alternativelly, a less intrusive option is to defined named getters, to call std::get with a defined argument:


``` C++
// Define functions called x, y, z, that call std::get with arguments 0, 1 and 2
NAMED_GETTERS(x, y, z)

int main ()
{
    std::tuple<double, double, double> point3d;

    x(point3d) = 10.0;
    y(point3d) = 20.0;
    z(point3d) = 30.0;

    return 0;
}
```

<br>

### Print

Some useful printing utilities.

By creating a handy::Print you can define how to print stuff. Also supports stl container printing:

``` C++
handy::Print printer(", ", "");   // Output to std::cout, separated by ", " and without line break ("")

printer(10, 20, 30);  // "10, 20, 30"

printer(std::vector<int>{10, 20, 30});  // Same as bove


std::ofstream myFile("myFile.txt");

handy::Print filePrinter(myFile, ", ", "");   // Output set to 'myFile'

filePrinter(10, 20, 30);    // Print to 'myFile'

filePrinter(std::cout, 10, 20, 30);  // Here the output is to std::cout, not to 'myFile'

filePrinter(10, 20, 30);    // Print to 'myFile' again
```


You can also use the direct function handy::print():

``` C++
handy::print(10, 20, 30);   // Exactly the same as handy::Print(std::cout, " ", "\n").operator()

std::ofstream myFile("myFile.txt");

handy::print(myFile, 10, 20, 30);   // Print to 'myFile' -> "10, 20, 30\n"

handy::print(myFile, std::vector<int>{10, 20, 30}); // Same as above
```

<br>


### Random

Utilities for sampling random numbers.

By creating a handy::RandDouble or handy::RandInt you can easily generate random numbers inside a given range:

``` C++
handy::RandInt randInt(0);  // If the seed is not given, a random seed is picked

int i1 = randInt(0, 10);
int i2 = randInt(10);     // Same as above


handy::RandDouble randDouble;

double d1 = randDouble();           // Random double in [0, 1]
double d2 = randDouble(0.0, 10.0);
double d3 = randDouble(10.0);       // Same as above


// You can also use the handy::Rand, which is a template
// For floating point types it is the same as handy::RandDouble and
// for integers handy::RandInt. Anything else will give you an error
handy::Rand<float> randFloat;

float f1 = randFloat(0.0, 10.0);
```


Also, you can use the handy::rand(int, int, unsigned int) or  handy::rand(double, double, unsigned int) directly:

``` C++
int i1 = handy::rand(0, 10);    // Argument is int -> call handy::RandInt

double d1 = handy::rand(0.0, 10.0);    // Argument is double -> call handy::RandDouble

double d2 = handy::rand(0.0, 10.0, 0);  // Seed is given. If called twice, it will give the same result
```


