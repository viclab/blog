title: 单例模式的应用
date: 2015-12-17 09:33:32
tags:
---
# 单例模式
  从设计模式的角度上来看，单例模式是最为常见也应用最广的模式。
它的原理其实很简单，就是在整个作用域里只允许也只有一个实例在运行。
引用《设计模式》对单例模式意图的描述，就是
> 保证一个类只有一个实例，并提供一个访问它的全局访问点
    
# 单例实现
  通常实现一个单例很简单，如下：
```cpp
class Singleton {
public:
    static Singleton* Instance() {
        if (!_instance) {
            _instance = new Singleton;
        }
    }
private:
    static Singleton* _instance;
};
```
  其实就是将类构造函数私有化，使不能直接调用构造函数来创建实例，而是通过特定的函数来获取实例。
但这样通常不够灵活，因为在项目中，需要用到单例模式的地方可能很多，如果每个用到的地方都写一个
单例，很不方便。所以，有两种方法来使得大量使用单例是更加便利：
## 继承
  可以给单例作为一个超类，需要用到单例的类继承自这个超类，然后
  <待续>

## 泛型
  将单例模式定义为一个泛型，需要用到的地方，直接具象化为需要用到的单例，这种方式比较。
而且很灵活，给单例作为一种特性，如果需要则具象化需要这种特性的类。这样，将类于单例实现完全分开。

```cpp
template <class T>                                                                                                                  
class Singleton : public NonCopyable {
 private:
     static T & instance;
    // include this to provoke instantiation at pre-execution time
    static void use(T const &) {}
    static T & get_instance() {
        static T t;
        // refer to instance, causing it to be instantiated (and
        // initialized at startup on working compilers)
        // assert(! detail::singleton_wrapper<T>::m_is_destroyed);
        use(instance);
        return static_cast<T &>(t);
    }   
 public:
    static T & get_mutable_instance() {
        return get_instance();
    }   
    static const T & get_const_instance(){
        return get_instance();
    }   
};

template<class T>
T& Singleton<T>::instance = Singleton<T>::get_instance();

}
```
在需要用到单例的类A中可以这样来使A支持单例特性
```cpp
typedef Singleton<A> ASingleton;
```

```
  
