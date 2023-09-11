---
title: Ray tracing in one weekend笔记
date: 2022-10-09
index_img: "/img/bg/RayTracing_weekend.jpg"
tags: [Ray Tracing]
categories: 
   -[Ray Tracing 笔记]
---

Ray tracing in one weekend笔记
<!-- more -->

# 参考资料

[Ray Tracing in one weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html)

[中文翻译](https://zhuanlan.zhihu.com/p/128582904)

# 输出图像

使用ppm格式

![](/article_img/2022-09-26-14-26-50.png)

显示ppm格式的图像使用：[OpenSeeIt](https://link.zhihu.com/?target=http%3A//openseeit.sourceforge.net/)，生成图像需要在cmd切换到应用程序目录并执行命令：
```
Raytracing_in_one_weekend.exe > image.ppm
```

![](/article_img/2022-09-26-14-28-48.png)

# vec3类

这里就复习一下c++语法吧

## const关键字小结：

1. const修饰普通类型的变量，表示是一个常量，不能重新赋值
   ```c++
   const a = 10;
   b = a; //正确
   a = 7; //错误
   ```
2. const 修饰指针变量，"左定值，右定向，const修饰不变量"，就是修饰谁谁就不能被改变
   ```c++
   // const修饰指针
   int a = 8;
   int * const p = &a;
   *p = 9; //正确
   p = &b; //错误

   // const修饰指针内容
   const int *p = 9;

   // const修饰指针和指针指向的内容, 指针p和*p都不能改变
   int a = 8;
   const int * const p = &a;
   ```
3. const参数传递和函数返回值
   ```c++
   // const修饰传递的值
   void fun(const int a){
      cout<<a;
      // ++a;  是错误的，a 不能被改变
   }
   // const修饰传递的指针,防止指针被意外篡改
   void fun(int * const p){
      ...
   }
   ```
   自定义类型的参数传递，需要临时对象复制参数，对于临时对象的构造，需要调用构造函数，比较浪费时间，因此我们采取 const 外加引用传递的方法。

   ```c++
   void fun(const my_class& a){
      cout<<a.class_fun();
   }
   ```
4. const修饰类成员函数

   const 修饰类成员函数，其目的是防止成员函数修改被调用对象的值，如果我们不想修改一个调用对象的值，所有的成员函数都应当声明为 const 成员函数。

## 构造函数和运算符重载

```c++
   //构造函数
   vec3(double x,double y,double z):e{x,y,z}{}
   //运算符重载
   vec3 operator*=(int t){
      ...
   }
   inline vec3 operator*(const vec3& u, const vec3& v) {
      return vec3(u.e[0] * v.e[0], u.e[1] * v.e[1], u.e[2] * v.e[2]);
   }
```

## inline关键字

在 c/c++ 中，为了解决一些频繁调用的小函数大量消耗栈空间（栈内存）的问题，特别的引入了 inline 修饰符，表示为内联函数。

栈空间就是指放置程序的局部数据（也就是函数内数据）的内存空间。

在系统下，栈空间是有限的，假如频繁大量的使用就会造成因栈空间不足而导致程序出错的问题，如，函数的死循环递归调用的最终结果就是导致栈内存空间枯竭。

**inline就是把函数内容替换到函数调用处。**

**注意：inline函数里不能有while和switch，不能是递归函数**

## C++中的别名定义

```c++
// 使用别名来区分点和颜色，只是用来区分，不会有警告
using point3 = vec3;
using color = vec3;
```

# 光线，简单摄像机和背景

![](/article_img/2022-09-26-21-08-46.png)

```c++
color ray_color(const ray& r) {
	// 线性插值颜色
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);
   // blendedValue=(1−t)⋅startValue+t⋅endValue
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0);
}
```

![](/article_img/2022-09-26-21-01-37.png)

# 添加球面

绘制一个球心在（0，0，-1）处，半径为0.5的球。

主要在于判断球面和光线是否相交，Games101中提到的光线与隐式表面的求交问题，将光线方程带入球面方程，求t。

![](/article_img/2022-09-27-11-50-26.png)

```c++
bool hit_sphere(const point3& center, double radius, const ray& r) {

	//  t2b⋅b+2tb⋅(A−C)+(A−C)⋅(A−C)−r2=0

	vec3 oc = r.origin() - center;
	auto a = dot(r.direction(), r.direction());
	auto b = 2 * dot(r.direction(), oc);
	auto c = dot(oc, oc) - radius * radius;
	auto discriminant = b * b - 4 * a * c;
	return (discriminant > 0);
}
```

![](/article_img/2022-09-27-11-50-57.png)

**存在的问题**：若将球心放置在（0，0，1）渲染的结果和此时一样。

# 法线和多个物体

## 法线可视化

```c++
	if (t > 0.0) { // 如果与球面相交直接return颜色
		// 求得法线
		vec3 N = unit_vector(r.at(t) - vec3(0, 0, -1));
      // 将值映射到（0，1），尽管已经是单位向量，但是由于法线坐标会有负值，仍需要做映射
		return 0.5 * color(N.x() + 1, N.y() + 1, N.z() + 1); 
	}
```

![](/article_img/2022-09-27-13-10-42.png)

## hit_record结构体

记录光线与物体交点的信息：交点坐标，交点处法线，光线方程中的t值

以及判断光线是从哪个方向射向平面，从而**设定法线方向始终与光线入射方向相反**。

```c++
struct hit_record { // 记录了与物体交点的信息：交点坐标，交点处法线，光线方程中的t值
	point3 p;
	vec3 normal;
	double t;

	// 判断光线是从哪个方向射向平面
	bool front_face;

	inline void set_face_normal(const ray& r, const vec3& outward_normal) {
		// 保证法线始终与光线入射方向相反
		front_face = dot(r.direction(), outward_normal) < 0;
		// 设置normal
		normal = front_face ? outward_normal : -outward_normal;
	}
};
```

## hittable类

定义了一个虚函数，本书中只解决了与球面判断相交的问题，就是将线段代入球面方程看是否有解。

```c++
class hittable {
public:
   virtual bool hit(const ray& r, double t_min, double t_max, hit_record& rec)const = 0;
};
```
## sphere类

sphere类继承自hittable类，实现了hit函数，有两个变量：球心坐标和半径

## hittable_list类（Some New C++ Features）


## 渲染结果

在main函数中创建hittable_list对象world，表示场景信息，其中有一个大球一个小球（渲染结果如下），地面（大球）是绿色是因为这里的法线都是向上，观察小球的顶部也是绿色。

![](/article_img/2022-09-27-19-39-22.png)

# 抗锯齿

加大采样率，在每个像素中随机采样一百次，将结果平均作为该像素的颜色。

![](/article_img/2022-09-28-16-15-11.png)

![](/article_img/2022-09-29-15-43-18.png)
将相机部分的代码封装到了camera类。

# 漫反射材质

![](/article_img/2022-09-29-15-45-06.png)

![](/article_img/2022-09-29-15-45-22.png)

![](/article_img/2022-09-29-15-45-33.png)

方法1： 单位圆中随机  
方法2： 在单位圆上随机生成  
方法3： 直接生成反射方向  

方法1比方法2要更加真实，因为前者能让更多光线向相机方向反射，而后者更多光线会靠近法线方法，导致反射次数增加（阴影很黑），使用方法2明显渲染速度变快很多。

```c++
color ray_color(const ray& r, const hittable& world, int depth) {
	hit_record rec;

	if (depth <= 0)
		return color(0, 0, 0);

	if (world.hit(r,0.001,infinity,rec)){ // 如果与球面相交直接return颜色
		// 求得法线
		//return 0.5 * (rec.normal + color(1,1,1));

		/* 漫反射
      方法1：单位圆中随机
		point3 target = rec.p + rec.normal + random_in_unit_vector(); 
      方法3：直接生成反射方向
		point3 target = rec.p + random_in_hemisphere(); 
      方法2：在单位圆上随机生成
      */
		point3 target = rec.p + rec.normal + random_unit_vector(); 
		return 0.5 * ray_color(ray(rec.p, target - rec.p), world, depth - 1); 
	}
	// 线性插值颜色
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0);
}
```

color(1,1,1)表示白色，color(0,0,0)表示黑色，这里的0.5*ray_color(...)相当于每次光线衰减50%，就是变暗一半。

![](/article_img/2022-09-29-15-52-40.png)

# 金属材质

## 材质类

抽象类： material类  
继承类： metal类，lambertian类

![](/article_img/2022-09-30-14-48-32.png)

## 左侧球呈现镜面效果的原因：  
光线从相机射出，与球面相交，调用交点材质的scatter函数计算反射光线以及得到衰减率，**进入递归**，发现反射光线不与物体相交，即进行线性插值颜色，**递归结束**，得到的线性插值颜色与衰减率相乘（**这就是为什么左侧球与背景颜色相似但是稍暗一些的原因**），得到最终的颜色。

## 右侧球带有颜色的原因：  
就是因为其衰减率的RGB三个值不相同，导致有的颜色衰减的更多，就看起来有了某种颜色，而不是简单的变暗。


## 关键代码

```c++
color ray_color(const ray& r, const hittable& world, int depth) {
	hit_record rec;

	if (depth <= 0)
		return color(0, 0, 0);

	if (world.hit(r,0.001,infinity,rec)){ // 与球面相交
		ray scattered;
		color attenuation;   // 衰减率

      // 调用材质的scatter函数，得到该反射点的材质的衰减率和反射光线。
		if (rec.mat_ptr->scatter(r, rec, attenuation, scattered))
			return attenuation * ray_color(scattered, world, depth - 1);
		return color(0, 0, 0);
	}
	// 线性插值颜色
	vec3 unit_direction = unit_vector(r.direction());
	auto t = 0.5 * (unit_direction.y() + 1.0);
	return (1.0 - t) * color(1.0, 1.0, 1.0) + t * color(0.5, 0.7, 1.0);
}
```

# 模糊反射（磨砂镜面）

![](/article_img/2022-10-08-14-55-02.png)

反射光线加一个随机的向量（在单位圆中生成）

在金属材质类（metal）中加一个属性：磨砂系数（fuzz）
```c++
scattered = ray(rec.p, reflected + fuzz * random_in_unit_sphere());
```
![](/article_img/2022-10-08-14-54-52.png)

# 电解质（透明物体）

## 折射

![](/article_img/2022-10-09-14-06-32.png)

一次折射会让场景颠倒

## Schlick近似

现实世界中的玻璃, 发生折射的概率会随着入射角而改变——从一个很狭窄的角度去看玻璃窗, 它会变成一面镜子。

## 中空球

将球的半径设置为负值，几何形状不会改变，但是法相全部反转到内部。

```c++
world.add(make_shared<sphere>(point3(-1.0,    0.0, -1.0),  -0.4, material_left));
```

![](/article_img/2022-10-09-14-25-53.png)

# 相机

## 相机可视角度（Field of View）

这里的可视角度是竖直方向上的可视角度，实际水平也可以。有了 FOV 再根据宽高比就能求出视图的宽高值。

![](/article_img/2022-10-09-14-28-04.png)

## 相机的定位定向

![](/article_img/2022-10-09-14-51-30.png)

![](/article_img/2022-10-09-14-51-38.png)

```c++
auto w = unit_vector(lookfrom - lookat);
auto u = unit_vector(cross(vup, w));
auto v = cross(w, u);
```

这里就是要新建一个坐标系，经过以上代码的运算，得到一个新的坐标系：相机看向-w方向，相机的向上方向是v，还有一个水平的方向u。

改变camera类，可以设定相机的位置和相机看向的方向。

```c++
camera cam(point3(-2, 2, 1), point3(0, 0, -1), vec3(0, 1, 0), 90, aspect_ratio);
```

![](/article_img/2022-10-09-14-52-21.png)

# 景深

![](/article_img/2022-10-09-15-26-37.png)

# 常看常新

## 成像平面

之前在我的印象中光线追踪模型应该是下图这样的，摄像机和成像平面在整个场景之前，但其实成像平面在哪并没有什么关系，本例的成像平面就与物体在同一深度。

![](/article_img/2022-10-29-18-15-59.png)

仔细一想，成像平面的意义在于确定从相机发出的路径，所以如下图所示，只要成像平面的面积与平面到相机的距离成比例变化，其实发出的路径都是一样的，都在图示的锥体中。

![](/article_img/2022-10-29-18-20-06.png)

而FOV实际上也是在改变这个锥体，FOV越大，成像平面也就越大（平面到相机的垂直相同的情况下）。

关于FOV还有一个有趣的观察，在这个光线追踪例子中，FOV越小，最终得到的图像和真正将相机离物体很近拍出的图像一模一样。而在现实中（如下图）我们可以轻易的分辨出是从很近的地方拍照还是通过变焦从很远的地方拍照（会明显模糊），而在本例子中并不会产生模糊。原因在于现实中的相机焦距不是任意可调的，理论上来说，拍摄越远的物体，相机内部的成像平面也应该离透镜越远，而真实的相机大小是有限的，因此会产生模糊，在本文中的虚拟的透镜焦距确实是可以任意调整的，因此并不会模糊，就像在很近的距离拍摄的一样，物体又大又清晰。

![](/article_img/2022-10-29-13-44-32.png)
