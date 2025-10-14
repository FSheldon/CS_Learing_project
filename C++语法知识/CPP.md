1. 何为STL？ 常见的STL有哪些？
    STL 是 C++ 标准库的核心部分，提供了一套通用的模板类和函数。它包含三大核心组件：
    容器（Containers）：用于存储数据的数据结构
    迭代器（Iterators）：用于遍历容器的对象
    算法（Algorithms）：用于处理容器中数据的函数

    常用算法：
    sort, find, binary_search, reverse, unique, max, min 等
    
    auto it = sort (vec.begin(), vec.end());
    if(it!=vec.end()){
        cout << *it;
    }

    set/map 可以直接insert 和 erase 而其他的必须通过 it 调用
    eg. vec.insert(vec.begin(), 要插入的值);
        auto it = vec.find(vec.begin(), vec.end(), , [要查的值]);
        vec.erase(it);

    vector -- int[]                     1 1 n   连续内存空间的动态数组
    queue deque     两端操作友好
    stack
    (multi)map unordered_(multi)map     logn    红黑树（自平衡二叉搜索树）
    (multi)set unordered_(multi)set             红黑树（元素自动排序，不重复）
    list(双向链表 不支持随机访问) 中间操作友好
    priority_queue

2. 其数据结构、基本调用方法？

    vector<int>::iterator it;              // 普通迭代器
    vector<int>::const_iterator cit;       // 常量迭代器（只读）
    vector<int>::reverse_iterator rit;     // 反向迭代器
    
    // begin() 和 end()
    it = v.begin();    // 指向第一个元素
    it = v.end();      // 指向最后一个元素的下一位置
    
    // 反向迭代器
    for (rit = v.rbegin(); rit != v.rend(); rit++) {
        cout << *rit << " ";  // 输出: 5 4 3 2 1
    }
    
    // 迭代器运算
    it = v.begin();
    it++;              // 下一个元素
    it--;              // 上一个元素
    it += 2;           // 向后移动2个位置（随机访问迭代器）

    // 距离
    int dist = distance(v.begin(), v.end());  // 5
    
    // 前进
    it = v.begin();
    advance(it, 3);    // 前进3个位置

   *Usual Algorithm

    vector<int> v = {5, 2, 8, 1, 9, 3};
    
    // 排序
    sort(v.begin(), v.end());              // 升序
    sort(v.begin(), v.end(), greater<int>()); // 降序
    
    // 查找
    auto it = find(v.begin(), v.end(), 8);
    if (it != v.end()) {
        cout << "Found at position: " << distance(v.begin(), it) << endl;
    }
    
    // 二分查找（需要已排序）
    sort(v.begin(), v.end());
    bool found = binary_search(v.begin(), v.end(), 5);
    
    // 反转
    reverse(v.begin(), v.end());
    
    // 最大最小值
    int max_val = *max_element(v.begin(), v.end());
    int min_val = *min_element(v.begin(), v.end());
    
    // 去重（需要先排序）
    sort(v.begin(), v.end());
    v.erase(unique(v.begin(), v.end()), v.end());
    
    // 计数
    int cnt = count(v.begin(), v.end(), 5);
    
    // 累加
    int sum = accumulate(v.begin(), v.end(), 0);
    
    return 0;


3. 容器怎么用？模版怎么写？
    1.// 基本函数模板
    template <typename T>
    T maximum(T a, T b) {
        return (a > b) ? a : b;
    }

    // 使用
    int main() {
        cout << maximum(10, 20) << endl;           // int版本
        cout << maximum(3.14, 2.71) << endl;       // double版本
        cout << maximum<int>(10, 20) << endl;      // 显式指定类型
        return 0;
    }

    // 多个模板参数
    template <typename T1, typename T2>
    void printPair(T1 first, T2 second) {
        cout << first << ", " << second << endl;
    }

    // 使用
    printPair(10, "hello");      // T1=int, T2=const char*
    printPair(3.14, 'A');        // T1=double, T2=char

    2.// 基本类模板
    template <typename T>
    class Box {
    private:
        T value;
        
    public:
        Box(T v) : value(v) {}
        
        T getValue() {
            return value;
        }
        
        void setValue(T v) {
            value = v;
        }
    };

    // 使用
    int main() {
        Box<int> intBox(100);
        Box<string> strBox("Hello");
        
        cout << intBox.getValue() << endl;
        cout << strBox.getValue() << endl;
        
        return 0;
    }

    3.自定义容器类模板
    template <typename T>
    class MyVector {
    private:
        T* data;
        int capacity;
        int length;
        
    public:
        // 构造函数
        MyVector() : data(nullptr), capacity(0), length(0) {}
        
        // 析构函数
        ~MyVector() {
            delete[] data;
        }
    
        // 添加元素
        void push_back(const T& value) {
            if (length >= capacity) {
                // 扩容
                int newCapacity = (capacity == 0) ? 1 : capacity * 2;
                T* newData = new T[newCapacity];
                
                // 复制旧数据
                for (int i = 0; i < length; i++) {
                    newData[i] = data[i];
                }
                
                delete[] data;
                data = newData;
                capacity = newCapacity;
            }
            
            data[length++] = value;
        }
    
        // 访问元素
        T& operator[](int index) {
            return data[index];
        }
        
        // 获取大小
        int size() const {
            return length;
        }
        
        // 判断是否为空
        bool empty() const {
            return length == 0;
        }
    };

    // 使用
    int main() {
        MyVector<int> vec;
        vec.push_back(10);
        vec.push_back(20);
        vec.push_back(30);
        
        for (int i = 0; i < vec.size(); i++) {
            cout << vec[i] << " ";
        }
        
        return 0;
    }

    4.// 通用模板
    template <typename T>
    class Printer {
    public:
        void print(T value) {
            cout << "Value: " << value << endl;
        }
    };

    // 特化版本（针对bool类型）
    template <>
    class Printer<bool> {
    public:
        void print(bool value) {
            cout << "Boolean: " << (value ? "true" : "false") << endl;
        }
    };

    // 使用
    int main() {
        Printer<int> intPrinter;
        Printer<bool> boolPrinter;
        
        intPrinter.print(42);      // 输出: Value: 42
        boolPrinter.print(true);   // 输出: Boolean: true
        
        return 0;
    }

    5. 泛型栈
    #include <iostream>
    #include <stdexcept>
    using namespace std;

    template <typename T>
    class Stack {
    private:
        T* data;
        int capacity;
        int topIndex;
        
    public:
        Stack(int size = 10) : capacity(size), topIndex(-1) {
            data = new T[capacity];
        }
        
        ~Stack() {
            delete[] data;
        }
        
        void push(const T& value) {
            if (topIndex >= capacity - 1) {
                throw overflow_error("Stack overflow");
            }
            data[++topIndex] = value;
        }
        
        T pop() {
            if (empty()) {
                throw underflow_error("Stack underflow");
            }
            return data[topIndex--];
        }
            
            T& top() {
                if (empty()) {
                    throw underflow_error("Stack is empty");
                }
                return data[topIndex];
            }
            
            bool empty() const {
                return topIndex == -1;
            }
            
            int size() const {
                return topIndex + 1;
            }
        };

    // 使用
    int main() {
        Stack<int> intStack;
        intStack.push(10);
        intStack.push(20);
        intStack.push(30);
        
        cout << "Top: " << intStack.top() << endl;
        cout << "Pop: " << intStack.pop() << endl;
        cout << "Size: " << intStack.size() << endl;
        
        Stack<string> strStack;
        strStack.push("Hello");
        strStack.push("World");
        
        cout << strStack.pop() << endl;
        
        return 0;
    }

    #include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
#include <map>
#include <string>
#include <unordered_set>
#include <functional>

    using namespace std;

    class Test{
        private:
            vector<int> vec;
            deque<float> dq;
            queue<char> q;
            stack<double> st;
            map<int, Test*> mp;
            unordered_set<int> us;
            priority_queue<int,vector<int>, greater<int>> minHeap; //小顶堆
            priority_queue<int> maxHeap; //大顶堆

    public:
        void Play_Vector(){
            vec = vector<int>(100);
            vec.push_back(1);
            vec.pop_back();
            vec.size();
            vec.clear();
            vec.empty();
            vec.resize(10,0);
            sort(vec.begin(), vec.end());
            reverse(vec.begin(), vec.end());
            auto it = find(vec.begin(), vec.end(), 1);
            auto it2 = lower_bound(vec.begin(), vec.end(), 1);
            auto it3 = upper_bound(vec.begin(), vec.end(), 1);
            auto it4 = unique(vec.begin(), vec.end());
            int max_val = *max_element(vec.begin(), vec.end());
            int min_val = *min_element(vec.begin(), vec.end());

            vec.insert(vec.begin(), 30);
            auto it = find(vec.begin(), vec.end(), 30);
            vec.erase(it);
            vec.insert(it, 2);
        }
};


4. lamba表达式
    当然可以！C++ 中的 **lambda 表达式**（lambda expression）是 C++11 引入的一个强大特性，让你可以**在代码中直接定义匿名函数**（即没有名字的函数），特别适合用在 STL 算法、回调、多线程等场景。

    下面我们从 **语法、捕获、用法、原理** 四个方面，**由浅入深、详细清晰地讲解**。

    ---

    ## 一、基本语法：像写函数，但更简洁

    ### ✅ 最简形式：
    ```cpp
    []() { /* 函数体 */ }
    ```

    ### ✅ 完整语法（带返回类型）：
    ```cpp
    [capture](parameters) -> return_type { body }
    ```

    ### 🔍 各部分含义：

    | 部分 | 说明 |
    |------|------|
    | `[capture]` | **捕获列表**：决定 lambda 能访问哪些外部变量（重点！）|
    | `(parameters)` | 参数列表，和普通函数一样 |
    | `-> return_type` | 返回类型（可省略，编译器会自动推导）|
    | `{ body }` | 函数体 |

    ---

    ## 二、捕获列表（Capture）—— 最关键的部分！

    lambda 可以“捕获”外部作用域的变量，方式有以下几种：

    ### 1. **不捕获任何变量**
    ```cpp
    []() { cout << "Hello"; }
    ```

    ### 2. **按值捕获（copy）**
    ```cpp
    int x = 10;
    [=]() { cout << x; }   // 捕获所有外部变量（按值）
    [x]() { cout << x; }   // 只捕获 x（按值）
    ```
    - 捕获的是 **x 的副本**，修改不会影响外部 x。
    - 默认是 **const** 的（不能修改副本），除非加 `mutable`。

    ### 3. **按引用捕获**
    ```cpp
    int x = 10;
    [&]() { x = 20; }      // 捕获所有外部变量（按引用）
    [&x]() { x = 20; }     // 只捕获 x（按引用）
    ```
    - 直接操作 **外部变量本身**，修改会生效。
    - ⚠️ 注意：**不要返回引用捕获的 lambda**（悬空引用！）

    ### 4. **混合捕获**
    ```cpp
    int x = 1, y = 2;
    [=, &y]() { x = 10; y = 20; }  // x 按值，y 按引用
    [&, x]() { x = 10; y = 20; }   // y 按引用，x 按值（覆盖默认）
    ```

    ### 5. **捕获 this（类成员函数中）**
    ```cpp
    class MyClass {
        int val = 42;
    public:
        void foo() {
            auto f = [this]() { cout << val; }; // 捕获 this 指针
            auto g = [*this]() { ... };         // C++17：按值捕获整个对象
        }
    };
    ```

    ---

    ## 三、实际使用场景（超实用！）

    ### ✅ 1. 用在 STL 算法中（最常见！）

    ```cpp
    std::vector<int> v = {1, 2, 3, 4, 5};

    // 找出第一个大于 3 的数
    auto it = std::find_if(v.begin(), v.end(), [](int x) { return x > 3; });

    // 排序：按绝对值
    std::sort(v.begin(), v.end(), [](int a, int b) {
        return abs(a) < abs(b);
    });

    // 累加：只加偶数
    int sum = std::accumulate(v.begin(), v.end(), 0,
        [](int acc, int x) { return x % 2 == 0 ? acc + x : acc; });
    ```

    ### ✅ 2. 作为回调函数

    ```cpp
    void setTimeout(std::function<void()> callback, int ms) {
        // 模拟异步
        std::thread([callback, ms]() {
            std::this_thread::sleep_for(std::chrono::milliseconds(ms));
            callback();
        }).detach();
    }

    // 调用
    int count = 0;
    setTimeout([&]() { count++; }, 1000); // 1秒后 count++
    ```

    ### ✅ 3. 多线程

    ```cpp
    std::thread t([]() {
        std::cout << "Hello from thread!\n";
    });
    t.join();
    ```

    ---

    ## 四、高级特性

    ### 🔸 1. `mutable`：允许修改按值捕获的变量

    ```cpp
    int x = 10;
    auto f = [=]() mutable {
        x = 20;  // 允许修改副本（但不影响外部 x）
        cout << x; // 输出 20
    };
    f();
    cout << x; // 仍是 10
    ```

    ### 🔸 2. 泛型 lambda（C++14）

    ```cpp
    auto add = [](auto a, auto b) { return a + b; };
    cout << add(1, 2);     // 3
    cout << add(1.5, 2.5); // 4.0
    ```
    > 等价于模板函数：
    > ```cpp
    > template<typename T, typename U>
    > auto add(T a, U b) { return a + b; }
    > ```

    ### 🔸 3. 返回类型推导（C++14 起更强大）

    ```cpp
    auto f = [](int x) { return x * 2; }; // 自动推导返回 int
    ```

    ---

    ## 五、lambda 的本质：编译器生成的匿名类

    编译器会把 lambda 转换成一个**匿名的函数对象类（functor）**：

    ```cpp
    // 你写的：
    auto f = [x](int y) { return x + y; };

    // 编译器生成类似：
    class __lambda_1 {
        int x;  // 按值捕获的变量成为成员
    public:
        __lambda_1(int x) : x(x) {}
        auto operator()(int y) const { return x + y; }
    };
    ```

    所以 lambda 本质上是一个**重载了 `operator()` 的对象**，可以像函数一样调用。

    ---

    ## 六、常见陷阱 & 注意事项

    | 问题 | 说明 |
    |------|------|
    | **悬空引用** | `[&x]` 捕获局部变量，但 lambda 生命周期比 x 长 → 未定义行为！ |
    | **忘记 mutable** | 按值捕获后想修改副本，必须加 `mutable` |
    | **性能开销** | 按值捕获大对象会拷贝，考虑用 `const&` 或移动语义 |
    | **不能递归** | lambda 不能直接调用自己（除非用 `std::function` 包装） |

    ---

    ## ✅ 总结：lambda 的核心价值

    | 优势 | 说明 |
    |------|------|
    | **简洁** | 无需单独定义函数 |
    | **局部性** | 逻辑紧贴使用处，提高可读性 |
    | **灵活捕获** | 按需访问外部变量 |
    | **高性能** | 编译期生成，无虚函数开销 |

    > 💡 **最佳实践**：  
    > - 简单逻辑 → 用 lambda  
    > - 复杂/复用逻辑 → 写普通函数或函数对象  
    > - 小心引用捕获的生命周期！

    ---

    希望这份详解能让你彻底掌握 C++ lambda！如果还有疑问（比如和 `std::function` 的关系、如何存储 lambda 等），欢迎继续问 😊

5.
    非常好的问题！`dynamic_cast` 和 `static_cast` 是 C++ 中两个常用的**显式类型转换操作符**，但它们的**用途、安全性、运行时行为**完全不同。下面我们用**清晰对比 + 例子 + 使用场景**来彻底讲明白。

    ---

    ## 🔑 一句话总结区别

    > - **`static_cast`**：**编译时**检查的“常规”类型转换（如 `int ↔ double`、**有继承关系的指针/引用向上/向下转**，但**不安全**）。
    > - **`dynamic_cast`**：**运行时**检查的“安全”向下转型（**仅用于多态类型**，失败返回 `nullptr` 或抛异常）。

    ---

    ## 一、`static_cast` —— 编译时转换

    ### ✅ 支持的转换类型
    1. **基本类型之间**：`int ↔ double`、`void* ↔ T*` 等。
    2. **有继承关系的类指针/引用**：
    - **向上转型（upcast）**：派生类 → 基类（**总是安全**）。
    - **向下转型（downcast）**：基类 → 派生类（**不检查！可能危险**）。
    3. **显式调用构造函数/转换函数**。

    ### ⚠️ 特点
    - **编译时完成**，无运行时开销。
    - **不做运行时类型检查**，如果向下转型错误，行为未定义（可能崩溃）。

    ### 🌰 例子

    ```cpp
    class Base { public: virtual ~Base() = default; };
    class Derived : public Base { public: void foo() { cout << "Derived"; } };

    int main() {
        // 1. 基本类型转换
        double d = 3.14;
        int i = static_cast<int>(d); // 3

        // 2. 向上转型（安全）
        Derived d_obj;
        Base* b1 = static_cast<Base*>(&d_obj); // OK

        // 3. 向下转型（危险！）
        Base* b2 = new Base();
        Derived* d1 = static_cast<Derived*>(b2); // 编译通过！
        d1->foo(); // ❌ 未定义行为！b2 实际不是 Derived
    }
    ```

    > 💡 `static_cast` 向下转型就像“我相信程序员知道自己在做什么”。

    ---

    ## 二、`dynamic_cast` —— 运行时安全转换

    ### ✅ 支持的转换类型
    - **仅用于多态类型**（类必须有**虚函数**，通常是虚析构函数）。
    - **主要用于向下转型（基类 → 派生类）**。
    - 也可用于**交叉转型（cross cast）**（多重继承场景）。

    ### ⚠️ 特点
    - **运行时检查**对象的实际类型（通过 RTTI，Run-Time Type Information）。
    - **安全**：转换失败时：
    - 对**指针**：返回 `nullptr`
    - 对**引用**：抛出 `std::bad_cast` 异常
    - **有运行时开销**（查虚表、类型信息）。

    ### 🌰 例子

    ```cpp
    class Base { public: virtual ~Base() = default; };
    class Derived : public Base { public: void foo() { cout << "Derived"; } };

    int main() {
        // 情况1：实际是 Derived 对象
        Base* b1 = new Derived();
        Derived* d1 = dynamic_cast<Derived*>(b1);
        if (d1) d1->foo(); // ✅ 输出 "Derived"

        // 情况2：实际是 Base 对象
        Base* b2 = new Base();
        Derived* d2 = dynamic_cast<Derived*>(b2);
        if (d2) {
            d2->foo(); // ❌ 不会执行
        } else {
            cout << "Cast failed!"; // ✅ 输出这个
        }

        // 引用版本（失败会抛异常）
        try {
            Base& b3 = *b2;
            Derived& d3 = dynamic_cast<Derived&>(b3);
        } catch (const std::bad_cast& e) {
            cout << "Bad cast!"; // ✅ 捕获异常
        }
    }
    ```

    > 💡 `dynamic_cast` 就像“让我先确认一下，再决定能不能转”。

    ---

    ## 三、关键对比表

    | 特性 | `static_cast` | `dynamic_cast` |
    |------|---------------|----------------|
    | **检查时机** | 编译时 | 运行时 |
    | **运行时开销** | 无 | 有（RTTI） |
    | **是否需要虚函数** | ❌ 不需要 | ✅ 必须（多态类型） |
    | **向下转型安全性** | ❌ 不安全（可能未定义行为） | ✅ 安全（失败返回 `nullptr`） |
    | **适用类型** | 基本类型、继承类、void* 等 | **仅多态类的指针/引用** |
    | **失败行为** | 编译失败（类型不兼容）或运行时崩溃 | 指针→`nullptr`，引用→`bad_cast` |

    ---

    ## 四、什么时候用哪个？

    ### ✅ 用 `static_cast` 当：
    - 转换基本类型（`int` → `double`）。
    - **向上转型**（派生类 → 基类）—— 虽然可以隐式转换，但显式写更清晰。
    - 你**100% 确定**向下转型是安全的（比如你自己刚创建的对象）。
    - 性能敏感，不能有 RTTI 开销。

    ### ✅ 用 `dynamic_cast` 当：
    - **不确定基类指针实际指向什么类型**（比如从容器中取出的基类指针）。
    - 需要**安全地向下转型**。
    - 处理**多态对象**，且愿意付出 RTTI 开销。

    ---

    ## 五、常见误区

    | 误区 | 正确理解 |
    |------|--------|
    | “`dynamic_cast` 可以转任何类型” | ❌ 只能用于**多态类型**（有虚函数的类） |
    | “`static_cast` 向下转型会检查” | ❌ 完全不检查，危险！ |
    | “RTTI 开销很大，永远别用 `dynamic_cast`” | ⚠️ 在需要安全转型时，这点开销值得；否则用 `static_cast` 或设计避免转型 |

    ---

    ## 六、最佳实践建议

    1. **优先设计避免类型转换**（比如用虚函数代替向下转型）。
    2. 如果必须向下转型：
    - 能确定类型 → `static_cast`
    - 不能确定类型 → `dynamic_cast`
    3. **确保基类有虚析构函数**（否则 `dynamic_cast` 不能用，且 delete 基类指针会未定义行为）。

    ```cpp
    // 好的设计：用虚函数，避免转型
    class Shape {
    public:
        virtual ~Shape() = default;
        virtual void draw() = 0;
    };

    class Circle : public Shape {
    public:
        void draw() override { /* 画圆 */ }
    };

    // 使用
    Shape* s = new Circle();
    s->draw(); // ✅ 直接调用，无需知道具体类型
    ```

    ---

    ## ✅ 总结

    | 转换 | 本质 | 安全性 | 性能 | 适用场景 |
    |------|------|--------|------|--------|
    | `static_cast` | 编译时“信任程序员”的转换 | ❌ 向下转型不安全 | ⚡ 快 | 基本类型、确定安全的继承转换 |
    | `dynamic_cast` | 运行时“验证类型”的转换 | ✅ 安全 | 🐢 有开销 | 多态类型的安全向下转型 |

    > 💡 **记住**：  
    > - **`static_cast` = “我相信你”**  
    > - **`dynamic_cast` = “让我验证一下”**

    希望这下彻底清楚啦！如有具体代码场景，欢迎继续讨论 😊

6. string

7. thred