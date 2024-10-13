
# 先写需要指定的模板参数，再把能推导出来的模板参数放在后面

```cpp
template <typename DstT, typename SrcT> DstT c_style_cast(SrcT v) // 模板参数 DstT 需要人肉指定，放前面。
{
return (DstT)(v);
}

int v = 0;
float i = c_style_cast<float>(v); // 形象地说，DstT 会先把你指定的参数吃掉，剩下的就交给编译器从函数参数列表中推导啦
```


还有一个作用是作为常数出现。所以整型模板参数最基本的用途，也是定义一个常数。例如这段代码的作用：
```cpp
template <typename T, int Size> struct Array
{
    T data[Size];
};

Array<int, 16> arr;

template <int i> class A
{
public:
    void foo(int)
    {
    }
};
template <uint8_t a, typename b, void*c> class B {};
template <bool, void (*a)()> class C {};
template <void (A<3>::*a)(int)> class D {};

template <int i> int Add(int a) // 当然也能用于函数模板
{
    return a + i;
}

void foo()
{
    A<5> a;
    B<7, A<5>, nullptr> b; // 模板参数可以是一个无符号八位整数，可以是模板生成的类；可以是一个指针。
    C<false, &foo> c;      // 模板参数可以是一个bool类型的常量，甚至可以是一个函数指针。
    D<&A<3>::foo> d;       // 丧心病狂啊！它还能是一个成员函数指针！
    int x = Add<3>(5);     // x == 8。因为整型模板参数无法从函数参数获得，所以只能是手工指定啦。
}

template <float a> class E {}; // ERROR: 别闹！早说过只能是整数类型的啦！
```
据类型执行不同代码
```cpp
c:
struct Variant
{
    union
    {
        int x;
        float y;
    } data;
    uint32 typeId;
};

Variant addFloatOrMulInt(Variant const*a, Variant const* b)
{
    Variant ret;
    assert(a->typeId == b->typeId);
    if (a->typeId == TYPE_INT)
    {
        ret.x = a->x *b->x;
    }
    else
    {
        ret.y = a->y + b->y;
    }
    return ret;
}
define BIN_OP(type, a, op, b, result) (*(type *)(result)) = (*(type const *)(a)) op (*(type const*)(b))
void doDiv(void* out, void const*data0, void const* data1, DATA_TYPE type)
{
    if(type == TYPE_INT)
    {
        BIN_OP(int, data0, *, data1, out);
    }
    else
    {
        BIN_OP(float, data0, +, data1, out);
    }
}
宏 void*
define BIN_OP(type, a, op, b, result) (*(type *)(result)) = (*(type const *)(a)) op (*(type const*)(b))
void doDiv(void*out, void const*data0, void const* data1, DATA_TYPE type)
{
    if(type == TYPE_INT)
    {
        BIN_OP(int, data0,*, data1, out);
    }
    else
    {
        BIN_OP(float, data0, +, data1, out);
    }
}
```
当模板实例化时提供的模板参数不能匹配到任何的特化形式的时候，它就会去匹配类模板的“原型”形式。
```cpp
template <typename T>
class RemovePointer<T*>
{
public:
    // 如果是传进来的是一个指针，我们就剥夺一层，直到指针形式不存在为止。
    // 例如 RemovePointer<int**>，Result 是 RemovePointer<int*>::Result，
    // 而 RemovePointer<int*>::Result 又是 int，最终就变成了我们想要的 int，其它也是类似。
    typedef typename RemovePointer<T>::Result Result;
};

struct A;
template <typename T> struct B;
template <typename T> struct X {
    typedef X<T> TA; // 编译器当然知道 X<T> 是一个类型。
    typedef X    TB; // X 等价于 X<T> 的缩写
    typedef T    TC; // T 不是一个类型还玩毛

    // ！！！注意我要变形了！！！
    class Y {
        typedef X<T>     TD;          // X 的内部，既然外部高枕无忧，内部更不用说了
        typedef X<T>::Y  TE;          // 嗯，这里也没问题，编译器知道Y就是当前的类型，
                                      // 这里在VS2015上会有错，需要添加 typename，
                                      // Clang 上顺利通过。
        typedef typename X<T*>::Y TF; // 这个居然要加 typename！
                                      // 因为，X<T*>和X<T>不一样哦，
                                      // 它可能会在实例化的时候被别的偏特化给抢过去实现了。
    };
    
    typedef A TG;                   // 嗯，没问题，A在外面声明啦
    typedef B<T> TH;                // B<T>也是一个类型
    typedef typename B<T>::type TI; // 嗯，因为不知道B<T>::type的信息，
                                    // 所以需要typename
    typedef B<int>::type TJ;        // B<int> 不依赖模板参数，
                                    // 所以编译器直接就实例化（instantiate）了
                                    // 但是这个时候，B并没有被实现，所以就出错了
};

template <typename T> class X      {};
template <typename T> class X <T*> {};
//                            ^^^^ 注意这里
必须要符合原型X的基本形式：那就是只有一个模板参数。这也是为什么DoWork尝试以template <> struct DoWork<int, int>的形式偏特化的时候，编译器会提示模板实参数量过多
```
```cpp
template <typename... Ts, typename U> class X {};              // (1) error!
template <typename... Ts>             class Y {};              // (2)
template <typename... Ts, typename U> class Y<U, Ts...> {};    // (3)
template <typename... Ts, typename U> class Y<Ts..., U> {};    // (4) error!
(3)和(1)不同，它并不是模板的原型，它只是Y的一个偏特化 偏特化时，模板参数列表并不代表匹配顺序，它们只是为偏特化的模式提供的声明，也就是说，它们的匹配顺序，只是按照<U, Ts...>来，而之前的参数只是告诉你Ts是一个类型列表，而U是一个类型，排名不分先后。
```
```cpp
include <type_traits>

template <typename T> T CustomDiv(T lhs, T rhs) {
    // Custom Div的实现
}

template <typename T, bool IsFloat = std::is_floating_point<T>::value> struct SafeDivide {
    static T Do(T lhs, T rhs) {
        return CustomDiv(lhs, rhs);
    }
};

template <typename T> struct SafeDivide<T, true>{    // 偏特化A
    static T Do(T lhs, T rhs){
        return lhs/rhs;
    }
};

template <typename T> struct SafeDivide<T, false>{   // 偏特化B
    static T Do(T lhs, T rhs){
        return lhs;
    }
};

void foo(){
    SafeDivide<float>::Do(1.0f, 2.0f); // 调用偏特化A
    SafeDivide<int>::Do(1, 2);          // 调用偏特化B
}

include <complex>
include <type_traits>

template <typename T> T CustomDiv(T lhs, T rhs) {
    T v;
    // Custom Div的实现
    return v;
}

template <typename T, typename Enabled = std::true_type> struct SafeDivide {
    static T Do(T lhs, T rhs) {
        return CustomDiv(lhs, rhs);
    }
};

template <typename T> struct SafeDivide<
    T, typename std::is_floating_point<T>::type>{    // 偏特化A
    static T Do(T lhs, T rhs){
        return lhs/rhs;
    }
};

template <typename T> struct SafeDivide<
    T, typename std::is_integral<T>::type>{          // 偏特化B
    static T Do(T lhs, T rhs){
        return rhs == 0 ? 0 : lhs/rhs;
    }
};

void foo(){
    SafeDivide<float>::Do(1.0f, 2.0f); // 调用偏特化A
    SafeDivide<int>::Do(1, 2);          // 调用偏特化B
    SafeDivide<std::complex<float>>::Do({1.f, 2.f}, {1.f, -2.f});
}
```
对SafeDivide<int>
通过匹配类模板的泛化形式，计算默认实参，可以知道我们要匹配的模板实参是SafeDivide<int, true_type>

计算两个偏特化的形式的匹配：A得到<int, false_type>,和B得到 <int, true_type>

最后偏特化B的匹配结果和模板实参一致，使用它。

针对SafeDivide<complex<float>>
通过匹配类模板的泛化形式，可以知道我们要匹配的模板实参是SafeDivide<complex<float>, true_type>

计算两个偏特化形式的匹配：A和B均得到SafeDivide<complex<float>, false_type>

A和B都与模板实参无法匹配，所以使用原型，调用CustomDiv
