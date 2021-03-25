# Using STL Containers with Eigen

[TOC]

## 1  Eigen中使用STL containers时，有两种情况需要额外操作：

- **STL containers**实例化的类型中包含**fixed-size vectorizable Eigen types**的变量

- 一个类中包含有**fixed-size vectorizable Eigen types**

## 2  什么是fixed-size vectorizable Eigen types？

  **fixed-size verctorizable**: 有固定的大小并且是**16个字节**的整数倍

- [Eigen::Vector2d](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#ga6c206cbf6f8f3b74bc63ecd362fc2ad6)
- [Eigen::Vector4d](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#ga9b2fcb53776a2829871f8a49009bef0b)
- [Eigen::Vector4f](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#gae6a8e578d2848cc75f573c15a73bd9b4)
- [Eigen::Matrix2d](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#ga3b934095f8a2834e6cc27267427239d3)
- [Eigen::Matrix2f](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#ga36b8989b6aa63020139fc36bae6979e0)
- [Eigen::Matrix4d](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#ga31c5fac458c04196a36b36b5e51127ff)
- [Eigen::Matrix4f](https://eigen.tuxfamily.org/dox/group__matrixtypedefs.html#ga3a5de8dfef28d29aed525611e15a37e3)
- [Eigen::Affine3d](https://eigen.tuxfamily.org/dox/group__Geometry__Module.html#gab0c57680a4d0de53bc749378b0320175)
- [Eigen::Affine3f](https://eigen.tuxfamily.org/dox/group__Geometry__Module.html#ga3902f2f19737ec9f16189e218919c505)
- [Eigen::Quaterniond](https://eigen.tuxfamily.org/dox/group__Geometry__Module.html#ga5daab8e66aa480465000308455578830)
- [Eigen::Quaternionf](https://eigen.tuxfamily.org/dox/group__Geometry__Module.html#ga66aa915a26d698c60ed206818c3e4c9b)

## 3  特殊操作

- 必须要进行16字节的内存对齐，Eigen提供了对齐操作符：aligned_allocator
- 如果使用std::vector容器，必须包含#include<Eigen/StdVector>头文件

## 4  如何操作？

- **STL containers**实例化的类型中包含**fixed-size vectorizable Eigen types**的变量

  - 局部操作

    - std::map

      ```c++
      std::map<int, Eigen::Vector4f>;
      
      #####调整为#####
      std::map<int, Eigen::Vector4f, std::less<int>, Eigen::aligned_allocator<std::pair<const int, Eigen::Vector4f>>>;
      ```

    - std::vector

      ```c++
      std::vector<Eigen::Vector4f>;
      
      #####调整为#####
      #include<Eigen/StdVector>
      std::vector<Eigen::Vector4f, Eigen::aligned_allocator<<Eigen::Vector4f>>;
      ```

  - 全局操作

    ```c++
    #include<Eigen/StdVector>
    EIGEN_DEFINE_STL_VECTOR_SPECIALIZATION(Matrix3d)
    
    std::vector<Eigen::Matrix3d>
    ```

- 一个类中包含有**fixed-size vectoriable Eigen types**

  - 直接添加 EIGEN_MAKE_ALIGNED_OPERATOR_NEW

    ```c++
    class Foo
    {
      ...
      Eigen::Vector2d v;
      ...
    };
    ...
    Foo *foo = new Foo;
    
    #######调整为######
    class Foo
    {
      ...
      Eigen::Vector2d v;
      ...
    public:
      EIGEN_MAKE_ALIGNED_OPERATOR_NEW
    };
    ...
    Foo *foo = new Foo;
    ```

  - 根据具体参数添加

    ```c++
    template<int n> class Foo
    {
      typedef Eigen::Matrix<float,n,1> Vector;
      enum { NeedsToAlign = (sizeof(Vector)%16)==0 };
      ...
      Vector v;
      ...
    public:
      EIGEN_MAKE_ALIGNED_OPERATOR_NEW_IF(NeedsToAlign)
    };
    ...
    Foo<4> *foo4 = new Foo<4>; // foo4 is guaranteed to be 128bit-aligned
    Foo<3> *foo3 = new Foo<3>; // foo3 has only the system default alignment
    ```

  - 私有结构体

    ```c++
    struct Foo_d
    {
      EIGEN_MAKE_ALIGNED_OPERATOR_NEW
      Vector2d v;
      ...
    };
    
    struct Foo {
      Foo() { init_d(); }
      ~Foo() { delete d; }
      void bar()
      {
        // use d->v instead of v
        ...
      }
    private:
      void init_d() { d = new Foo_d; }
      Foo_d* d;
    };
    ```

- 禁用Eigen::allocator

  ```c++
  class Foo
  {
    ...
    ###使用v向量时禁用Eigen对齐
    Eigen::Matrix<double,2,1,Eigen::DontAlign> v;
    ...
  };
  
  ```

  如果v赋值给内存对齐的vector,则Eigen对齐会重启

  ```c++
  void Foo::bar()
  {
    ###该操作会重启v的Eigen对齐功能
    Eigen::Vector2d av(v);
    // use av instead of v
    ...
    // if av changed, then do:
    v = av;
  }
  ```


## 5 参考

[Using STL Containers with Eigen](<https://eigen.tuxfamily.org/dox/group__TopicStlContainers.html>)

[Fixed-size vectorizable Eigen objects](<https://eigen.tuxfamily.org/dox/group__TopicFixedSizeVectorizable.html>)

[Structures Having Eigen Members](<https://eigen.tuxfamily.org/dox/group__TopicStructHavingEigenMembers.html>)

