@[TOC](目录)
# 简介
>gtest是一个跨平台的C++单元测试框架，由google公司发布。gtest提供了丰富的断言，供开发者对代码块进行白盒测试。

# 使用目的
- 测试代码逻辑是否正确。编译器只能检测出语法错误但是无法检测到 ==*逻辑错误*==，比如一个函数或类是否完成了期望的功能。
- gtest可以帮助我们判断代码 ==*设计得是否清晰合理*==。一块代码的逻辑越清晰，它的测试就可以设计得越简单。
- 方便并行开发。一个程序有不同模块相互耦合，某个模块未完成可能影响其他已完成模块的测试，这时可以利用gmock ==*隔离*== 这些模块，模拟未完成模块的功能，进而测试已完成模块的逻辑。
- 每完成一个模块就用gtest进行验证。这比完成了整个程序再查找bug轻松许多。

# 使用时机
- 使用gtest需要维护额外的测试代码，花费额外的时间，但是可以轻松验证各个模块的逻辑功能是否正确，保证程序整体的正确性。
- 使用手动测试比较快捷，但是测试不全面，而且有些跟其他模块相关的功能测试起来难度很大。
- 对于那些会长期维护的大项目像服务器，使用gtest验证逻辑是有必要的。
- 对于那些短期使用的，后期不会进行维护的，功能比较简单的程序，手动测试的效果会更好。

# 使用方法
## 使用流程
1. 包含必要的头文件：声明了待测试代码的头文件、"gtest/gtest.h"、"gmock/gmock.h"(使用模拟时会用到)。
2. 使用宏编写测试主体：在测试宏中调用断言宏完成单元测试，对于复杂的单元测试，可以使用测试夹具。
3. 调用 `::testing::InitGoogleTest(int argc, char**argv)` 用参数初始化测试，`RUN_ALL_TESTS()` 运行所有测试并输出结果到控制台或文件中(默认控制台)，这部分最好写在独立于被测试项目的控制台应用程序的主函数中，这样可以减少复杂度。
## 传入参数
1. 参数种类

|命令行|代码变量名|说明|
|:--:|:--:|:--|
|--gtest_list_tests|无|列举出所有测试名称但是不执行，测试名格式：==测试案例名.测试名==|
|--gtest_filter<br>=POSITIVE_PATTERNS[-NEGATIVE_PATTERNS]|::testing::FLAGS_gtest_filter(std::string)|	对执行的测试案例进行过滤，支持通配符： <br>? 单个字符<br>* 任意字符<br>- 排除，如，-a 表示除了a<br>: 取或，如，a:b 表示a或b<br>比如下面的例子：<br>./foo_test 没有指定过滤条件，运行所有案例<br>./foo_test --gtest_filter=* 使用通配符*，表示运行所有案例<br>./foo_test --gtest_filter=FooTest.* 运行所有“测试案例名称<br>testcase_name)”为FooTest的案例<br>./foo_test --gtest_filter=Null:Constructor 运行所有“测试案例名称<br>(testcase_name)”或“测试名称(test_name)”包含Null或Constructor的案例。<br>./foo_test --gtest_filter=-DeathTest. 运行所有非死亡测试案例。<br>./foo_test --gtest_filter=FooTest.*-FooTest.Bar 运行所有“测试案例名称(testcase_name)”为FooTest的案例，但是除了FooTest.Bar这个案例|
|--gtest_also_run_disabled_tests|::testing::FLAGS_gtest_also_run_disabled_tests(bool)|执行案例时，同时也执行被置为无效的测试案例或测试<br>设置无效测试案例或无效测试时，需要在要设置的测试案例或测试名前加上 *DISABLED_* 前缀|
|--gtest_repeat<br>=[COUNT]|::testing::FLAGS_gtest_repeat(int32_t)|设置案例重复运行次数<br>–gtest_repeat=-1 无限次数执行<br>–gtest_repeat=1000 --gtest_break_on_failure 重复执行1000次，并且在第一个错误发生时立即停止<br>–gtest_repeat=1000 --gtest_filter=FooBar 重复执行1000次测试案例名称为FooBar的案例|
|--gtest_shuffle|::testing::FLAGS_gtest_shuffle(bool)|随机运行测试|
|--gtest_random_seed<br>=[NUMBER]|::testing::FLAGS_gtest_random_seed(int32_t)|设置一个随机种子|
|--gtest_color<br>=(yes\|no\|auto)|::testing::FLAGS_gtest_color(std::string)|是否输出颜色，有三种选项，“yes”, "no","auto"|
|--gtest_brief=1|::testing::FLAGS_gtest_brief(bool)|只打印错误测试信息，默认全部打印|
|--gtest_print_time=0|::testing::FLAGS_gtest_print_time(bool)|不打印测试运行的时间信息，默认打印|
|--gtest_output=(json\|xml)<br>[:DIRECTORY_PATH\|:FILE_PATH]|::testing::FLAGS_gtest_output(std::string)|将结果输出到json或xml文件中<br>--gtest_output=xml: 不指定输出路径时，默认为案例当前路径<br>--gtest_output=xml:d:\ 指定输出到某个目录<br>--gtest_output=<br>xml:d:\foo.xml 指定输出到d:\foo.xml|
|--gtest_break_on_failure|::testing::FLAGS_gtest_break_on_failure(bool)|调试模式下，当案例失败时停止，方便调试|
|--gtest_throw_on_failure|::testing::FLAGS_gtest_throw_on_failure(bool)|当案例失败时以C++异常的方式抛出|
|--gtest_catch_exceptions=0|::testing::FLAGS_gtest_catch_exceptions(bool)|测试不再抓取异常，而是而是直接让程序报错，默认将异常视为测试失败|
2. 传入方式：
	- 通过cmd调用程序时传入。
	- 在代码中设置对应的值。
	- 利用系统环境变量(不常用)。
	- 在命令行中传入 *--help*  可以查看所有参数说明。
## 用法
### 最简单的单元测试
```cpp
// sample1.h
#ifndef GOOGLETEST_SAMPLES_SAMPLE1_H_
#define GOOGLETEST_SAMPLES_SAMPLE1_H_
// 返回n的阶乘
static int Factorial(int n)
{
	int result = 1;
	for (int i = 1; i <= n; i++) 
	{
		result *= i;
	}
  return result;
}
// 当n是质数时返回真
static bool IsPrime(int n)
{
	// Trivial case 1: small numbers
	if (n <= 1) 
		return false;
	// Trivial case 2: even numbers
	if (n % 2 == 0) 
		return n == 2;
	// Now, we have that n is odd and n >= 3.
	// Try to divide n by every odd number i, starting from 3
	for (int i = 3; ; i += 2) 
	{
	// We only have to try i up to the square root of n
		if (i > n/i) 
			break;
	// Now, we have i <= n/i < n.
	// If n is divisible by i, n is not prime.
		if (n % i == 0)
			return false;
	}
	// n has no integer factor in the range (1, n), and thus is prime.
	return true;
}
#endif  // GOOGLETEST_SAMPLES_SAMPLE1_H_
```
```cpp
// sample1_unittest.cc
// Step.1 包含必要头文件
#include "sample1.h"
#include "gtest/gtest.h"
// Step.2 编写测试主体
TEST(FactorialTest, Negative) {
  // This test is named "Negative", and belongs to the "FactorialTest"
  // test case.
  EXPECT_EQ(1, Factorial(-5));
  EXPECT_EQ(1, Factorial(-1));
  EXPECT_GT(Factorial(-10), 0);
  // <TechnicalDetails>
  //
  // EXPECT_EQ(expected, actual) is the same as
  //
  //   EXPECT_TRUE((expected) == (actual))
  //
  // except that it will print both the expected value and the actual
  // value when the assertion fails.  This is very helpful for
  // debugging.  Therefore in this case EXPECT_EQ is preferred.
  //
  // On the other hand, EXPECT_TRUE accepts any Boolean expression,
  // and is thus more general.
  //
  // </TechnicalDetails>
}

// Tests factorial of 0.
TEST(FactorialTest, Zero) {
  EXPECT_EQ(1, Factorial(0));
}

// Tests factorial of positive numbers.
TEST(FactorialTest, Positive) {
  EXPECT_EQ(1, Factorial(1));
  EXPECT_EQ(2, Factorial(2));
  EXPECT_EQ(6, Factorial(3));
  EXPECT_EQ(40320, Factorial(8));
}
```
```cpp
// main.cpp
// Step.3 初始化测试，执行所有测试
int main(int argc, char** argv)
{
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```
### 普通测试夹具
1. 属性：普通测试夹具是继承了 ==::testing::Test== 的类，可以保存对象和函数方法，让所有属于这个测试案例的测试都共享这些对象和函数方法。但是只是共享这些测试的代码而已，一个测试对数据的改动不会传递给下一个测试，因为一个测试的结果不应该影响到下一个测试，如果需要，那这两个测试应该合并为一个。
2. 用法：继承 ==::testing::Test== ，测试宏应使用 `TEST_F` ，测试案例名必须与夹具名相同，测试才能使用夹具中的对象和方法，夹具类可以重载 `SetUp` 和 `TearDown` 两个函数，这两个函数在每个测试开始前和结束后都会调用一次，用于初始化和释放资源。
3. 注意：夹具可以多重继承，使得夹具可以拥有多种属性。
4. 样例代码
```cpp
// func.h
class TestClass
{
public:
	TestClass(int num) { this->num = num; }
	~TestClass() {}
	// 偶数返回真，奇数返回假
	bool Check()
	{
		return num % 2 == 0 ? true : false;
	}
	int num;
};

// UniteTest.cpp
#include "func.h"
#include <gtest/gtest.h>
class TestFixture : public testing::Test
{
public:
    TestFixture() {}
    virtual ~TestFixture() {}

protected:
    // 测试案例中的每个测试开始前都会调用
    virtual void SetUp() override { testClass = new TestClass(2); }
    // 测试案例中的每个测试结束后都会调用
    virtual void TearDown() override { delete testClass; }
    TestClass* testClass;
};

TEST_F(TestFixture, test1)
{
	EXPECT_TRUE(testClass->Check());
}
```

### 模板测试夹具
1. 属性：模板测试夹具是一个继承了 ==::testing::Test== 的模板类，这种测试夹具可以用于测试同一个接口类(父类)的多种实现。
2. 用法:
	- 创建继承 ==::testing::Test== 的模板类，模板的类型是接口类的实现类型(子类)，因此夹具中要保存接口类的对象指针(可以指向所有子类对象)，然后根据不同情况选择下面两步。
	- 如果已知参数类型，那么在编写测试主体前先使用 `TYPED_TEST_SUITE(TestCaseName, TypeList)` 传入所有类型， 再使用 `TYPED_TEST(TestCaseName, TestName)` 编写测试主体。
	- 如果参数类型未知(假设你是接口的设计者，在编写单元测试时并不知道接口会被如何实现)，那么先使用 `TYPED_TEST_SUITE_P(TestCaseName)` 声明测试夹具，再使用 `TYPED_TEST_P(TestCaseName, TestName)` 编写测试主体，最后使用 `REGISTER_TYPED_TEST_SUITE_P(TestCaseName, TestList)` 枚举出测试的名字，此时单元测试已经开发完成，但是不会运行任何测试，等到确定了待测试的类型后，使用 `INSTANTIATE_TYPED_TEST_SUITE_P(Instancename, TestCaseName, Typelist)` 即可运行所有类型的测试。
4. 样例代码：
```cpp
// interface.h
// 待测试接口类
class CTest
{
public:
	virtual bool TestFunc() = 0;
};

class CA : public CTest
{
	virtual bool TestFunc() override { //... }
};
class CB : public CTest
{
	virtual bool TestFunc() override { //... }
} ;
```
```cpp
// UnitTest.h
#include "interface.h"
#include "gtest/gtest.h"
// 工厂函数
template<class T>
CTest* CreateFunc() { return new T(); }
template<class T>
class TestFixture : public ::testing::Test
{
protected:
	TestFixture() { pTest = CreateFunc<T>(); }
	virtual ~TestFixture () { delete pTest; }
	// 基类指针
	CTest* pTest;
};

//==========测试类型已知==========
// 声明要测试的类型
TYPED_TEST_SUITE(TestFixture, ::testing::Types<CA, CB>);
// 测试主体
TYPED_TEST(TestFixture, A)
{
// ...
}

//==========测试类型未知==========
// 声明测试夹具
TYPED_TEST_SUITE_P(TestFixture);
// 测试主体
TYPED_TEST_P(TestFixture, A)
{
// ...
}
TYPED_TEST_P(TestFixture, B)
{
// ...
}
// 枚举所有测试
REGISTER_TYPED_TEST_SUITE_P(TestFixture, A, B);
// 声明所有测试类型
INSTANTIATE_TYPED_TEST_SUITE_P(TestInstance, TestFixture, ::testing::Types<CA, CB>);
```
### 参数测试夹具
1. 属性：参数测试夹具继承了 ==::testing::TestWithParam== ，可以在类中调用 `GetParam()` 获取传入的参数，从而设置内部对象或函数的属性。
2. 用法：参数测试夹具的用法与测试类型未知的模板测试夹具类似。
	- 创建继承 ==::testing::TestWithParam== 的测试夹具类，在类中可以调用 `GetParam()` 获取参数，设置内部对象或函数的属性，然后使用 `TEST_P(TestCaseName, TestName)` 编写测试主体，最后使用 `INSTANTIATE_TEST_SUITE_P(Instancename, TestCaseName, Typelist)` 注册所有参数。
3. 样例代码：
```cpp
#include "interface.h"
#include "gtest/gtest.h"
// 函数指针
typedef CTest* CreateCTestPtr();
// 工厂函数
CTest* CreateA() { return new CA(); }
CTest* CreateB() { return new CB(); ]
// 参数测试夹具，参数类型是函数指针
class TestFixture : public ::testing::TestWithParam<CreateCTestPtr*>
{
public:
	virtual ~TestFixture() { delete pTest; }
	virtual void SetUp() override { pTest = (*GetParam())(); }
	virtual void TearDown() override 
	{ 
		delete pTest;
		pTest = nullptr;
	}
protected:
	CTest* pTest;
};
// 测试主体
TEST_P(TestFixture, A)
{
// ...
}
// 注册参数，参数是构造不同类型实例的函数
INSTANTIATE_TEST_SUITE_P(TestInstance, TestFixture, ::testing::Values(&CreateA, &CreateB));
```
4. 补充：
如果参数类型可以由多个变量组合而成如 `std::tuple<bool, int>` ，如果想覆盖所有测试路径，使用传统的注册参数方法势必会很麻烦，可以使用 `::testing::Combine` 函数，该函数会自动组合出参数中所有组合情况，以 `std::tuple<bool, int>` 为例，代码修改为如下形式：
```cpp
class TestFixture : public ::testing::TestWithParam<std::tuple<bool, int>>
// ...
INSTANTIATE_TEST_SUITE_P(TestInstance, TestFixture, 
	::testing::Combine(::testing::Bool(), ::testing::Values(1, 10)));
```
`::testing::Bool()` 会生成 true 和 false两种值，`::testing::Combine` 把  *{true，false}* 和 *{1， 10}* 这两个集合的值组合起来，相当于 
```cpp
::testing::Values(
	std::tuple<bool, int>(true, 1), 
	std::tuple<bool, int>(false, 1), 
	std::tuple<bool, int>(true, 10),
	std::tuple<bool, int>(false, 10)
)
```
## 宏定义总结
### 测试宏
1. `TEST(TestCaseName, TestName)`, `TestCaseName`表示测试案例名， `TestName`表示测试名，逻辑上相关的测试应该属于相同的测试案例，测试一个函数时，测试案例名可以是函数名，测试名可以是不同输入情况的名字；测试一个类时案例名可以是类名，测试名可以是不同函数的名字。这些名字必须是c++风格的合法字符串，不能带有下划线_。
2. `TEST_F(TestCaseName, TestName)`，与测试夹具配套使用，测试案例名必须是测试夹具的名字。
3. `TYPED_TEST(TestCaseName, TestName)`，与已知类型的模板测试夹具配套使用，在测试体中可以使用`TypeParam`指代模板参数类型，使用`TestFixture`指代夹具类类型。
4. `TYPED_TEST_P(TestCaseName, TestName)`，与未知类型的模板测试夹具配套使用，在测试体中可以使用`TypeParam`指代模板参数类型，使用`TestFixture`指代夹具类类型。
5. `TEST_P(TestCaseName, TestName)`，与参数夹具配套使用。
### 声明宏
1. `TYPED_TEST_SUITE(TestCaseName, TypeList)`， 声明模板测试夹具要测试的类型，TestCaseName是模板测试夹具的名字，TypeList代表待测试的类型，用 `::testing::Types<类型1, 类型2, ...>` 定义，在已知全部需要测试的类型时使用。
2. `TYPED_TEST_SUITE_P(TestCaseName)`， TestCaseName是模板测试夹具名，在不知道要测试的类型时使用。
3. `REGISTER_TYPED_TEST_SUITE_P(TestCaseName, TestName1, TestName2, ...)`，TestCaseName是模板测试夹具名，TestName是测试的名字，把所有测试的名字枚举出来。
4. `INSTANTIATE_TYPED_TEST_SUITE_P(Instancename, TestCaseName, Typelist)`，Instancename是测试实例名，TestCaseName是测试夹具名，Typelist代表待测试类型列表，用`::testing::Types<类型1, 类型2, ...>` 定义。
5. `INSTANTIATE_TEST_SUITE_P(Instancename, TestCaseName, Typelist)`, Instancename是测试实例名，TestCaseName是测试夹具名，Typelist代表参数列表，用`::testing::Values(参数1, 参数2, ...)`定义。
### 断言宏
- 区别：中断断言在失败后会立刻退出测试，不会执行后续代码，期望断言失败后会继续运行后续代码。
- 用法：一个测试中可以包含多个断言宏，所有断言均通过则测试通过。

|中断|期望|意义|
|:--:|:--:|:--:|
|ASSERT_TRUE(exp)|EXPECT_TRUE(exp)|验证exp为true|
|ASSERT_FALSE(exp)|EXPECT_FALSE(exp)|验证exp为false|
|ASSERT_EQ(val1, val2)|EXPECT_EQ(val1, val2)|val1 == val2|
|ASSERT_NE(val1, val2)|EXPECT_NE(val1, val2)|val1 != val2|
|ASSERT_LT(val1, val2)|EXPECT_LT(val1, val2)|val1 < val2|
|ASSERT_LE(val1, val2)|EXPECT_LE(val1, val2)|val1 <= val2|
|ASSERT_GT(val1, val2)|EXPECT_GT(val1, val2)|val1 > val2|
|ASSERT_GE(val1, val2)|EXPECT_GE(val1, val2)|val1 >= val2|
|ASSERT_STREQ(str1, str2)|EXPECT_STREQ(str1, str2)|str1 == str2|
|ASSERT_STRNE(str1, str2)|EXPECT_STRNE(str1, str2)|str1 != str2|
|ASSERT_STRCASEEQ(str1, str2)|EXPECT_STRCASEEQ(str1, str2)|(忽略大小写)str1 == str2|
|ASSERT_STRCASENE(str1, str2)|EXPECT_STRCASENE(str1, str2)|(忽略大小写)str1 != str2|
|SUCCEED()|SUCCEED()|直接返回测试成功|
|FAIL()|ADD_FAILURE()|直接返回测试失败|
|ASSERT_THROW(语句, 异常类型)|EXPECT_THROW(语句, 异常类型)|语句所指定的代码抛出给定的异常|
|ASSERT_ANY_THROW(语句)|EXPECT_ANY_THROW(语句)|语句所指定的代码抛出任何一种异常|
|ASSERT_NO_THROW(语句)|EXPECT_NO_THROW(语句)|语句所指定的代码不抛出任何异常|
|ASSERT_PRED1(pred1, val1)|ASSERT_PRED1(pred1, val1)|pred1(val1)返回true|
|ASSERT_PRED2(pred2, val1, val2)|ASSERT_PRED2(pred2, val1, val2)|pred2(val1, val2)返回true|
|ASSERT_PRED_FORMAT1(p_format1, val1)|EXPECT_PRED_FORMAT1(p_format1, val1)|p_format1(val1)成功|
|ASSERT_FLOAT_EQ(expected, actual)|EXPECT_FLOAT_EQ(expected, actual)|float几乎相等|
|ASSERT_DOUBLE_EQ(expected, actual)|EXPECT_DOUBLE_EQ(expected, actual)|double几乎相等|
|ASSERT_NEAR(val1,val2,abs_error)|EXPECT_NEAR(val1,val2,abs_error)|两个数之间的差不超过绝对值abs_error|
|ASSERT_DEATH(statement, regex)|EXPECT_DEATH(statement, regex)|程序挂了且给出的错误与指定错误一致|
|ASSERT_DEATH_IF_SUPPORTED(statement, regex)|EXPECT_DEATH_IF_SUPPORTED(statement, regex)|如果死亡测试支持，说明程序报的错误与给定错误一致|
|ASSERT_EXIT(statement, predicate, regex)|EXPECT_EXIT(statement, predicate, regex)|程序以指定的输出退出|
## gmock
1. 使用场景：当待测模块耦合了一个尚未完成或者运行环境非常复杂无法搭建的模块时，强行等待这个模块完成或者搭建环境都会耗费大量的时间，这时我们可以利用gmock模拟这个模块，将它的实际代码从我们要测试的模块从中隔离出来。
2. 用法：gmock的用法与类的继承差不多，直接看代码吧。
```cpp
class Tmp
{
public:
	virtual bool FuncA(int, int) = 0;
	virtual bool FuncB(int) = 0;
};
// 待测试模块
class TestClass
{
public:
	bool TestFunc(Tmp* pTmp)
	{
		if( pTmp->FuncA(1, 2) )
		{
			return pTmp->FuncB(3);
		}
		return false;
	}
};
// 模拟
class MockTmp : public Tmp
{
public:
	MOCK_METHOD2(FuncA, bool(int, int));
	MOCK_METHOD1(FuncB, bool(int));
};

TEST(MockCase, MockTest)
{
	MockTmp mock;
	EXPECT_CALL(mock, FuncA(::testing::Ne(0), ::testing::_))
	.With(::testing::Lt())
	.With(::testing::AllArgs(::testing::Lt()))
	.With(::testing::Args<0, 1>(::testing::Lt()))
	.Times(3)
	.WillOnce(::testing::Return(true));
	
	EXPECT_CALL(mock, FuncB(::testing::_))
	.WillOnce(::testing::Return(true));
	
	TestClass test;
	Tmp* pTmp = (Tmp*)&mock;
	EXPECT_TRUE(test.TestFunc(pTmp));
}
```
3. MOCK_METHOD*(函数名，A(B, C, ...))
*代表函数的参数个数，A表示函数返回值类型，B，C，...，代表函数的各个参数
4. EXPECT_CALL
原型：<br>`EXPECT_CALL(mock_object, method(matcher1, matcher2, ...))`<br>`.With(multi_argument_matcher)`<br>`.Times(cardinality)<br>.InSequence(sequences)`<br>`.After(expectations)`<br>`.WillOnce(action)`<br>`.WillRepeatedly(action)`<br>`.RetiresOnSaturation()`;
- EXPECT_CALL内的参数意义
  * `mock_object`表示Mock类的对象(mock)
  * `method`表示Mock的方法(Func)
  *  `matcher`是匹配器，可以用于定义函数方法参数的值，也可以判断输入的参数是否符合匹配器，匹配器给出的功能与断言类似，名称也很像，下面只列出其中一部分，可以参考gtest和gmock的文档。
 
	|匹配器|说明|
	|:--:|:--:|
	|`::testing::_`|不在乎输入参数是什么|
	|`::testing::Ne(val)`|参数不等于val|
	|`::testing::Eq(val)`|参数要与val相等|
	|`::testing::Lt(val)`|参数要小于val|
	|`::testing::Le(val)`|参数小于等于val|
	|`::testing::Gt(val)`|参数大于val|
	|`::testing::Ge(val)`|参数大于等于val|
	|...|...|

- 设置EXPECT_CALL的属性
  - .With(multi_argument_matcher)
 这个函数可以设置参数之间的大小关系，相等、从大到小或从小到大等。当对全部参数设置大小关系时（比如所有参数递增），可以使用下面两种等效方法：`.With(::testing::AllArgs(::testing::Lt()))`、`.With(::testing::Lt())`，可以注意到 `::testing::Lt()` 其实就是上面的匹配器，使用在.With中时，不需要填参数。如果想指定所有参数中的某些参数的关系可以使用 `::testing::Args<k1, k2, ..., kn>` 替换 `::testing::AllArgs` ，来指定n个参数的关系。
  - .Times(cardinality)
  这个函数可以设置被模拟的函数将被调用多少次。参数 *cardinality* 可以使用如下几个函数设置：
	|||函数方法|说明|
	||||:--:|:--:|
	| ``AnyNumber()`       |这个函数可以被调用任意次|
	| ``AtLeast(n)`        |这个函数最少被调用n次|
	| ``AtMost(n)`         |这个函数最多被调用n次|
	| ``Between(m, n)`     |这个函数被调用m到n次之间(包括m和n)|
	| ``Exactly(n)` or `n` |这个函数只能被调用n次|
    如果.Times缺省，gtest遵循下列原则：
    * 如果既没有设置 `.WillOnce` 也没有 `.WillRepeatedly`， 则视为调用了 `.Times(1)`；
    * 如果调用了 *n* 次 `WillOnce` (n >= 1) 且没有调用 `.WillRepeatedly`，则视为调用了`.Times(n)`；
    * 如果调用了 *n* 次`WillOnce` (n >= 0) 且调用了一次 `.WillRepeatedly`，则视为调用了`Times(AtLeast(n))`。
  - .InSequence(sequences)
  这个函数用来设置多个EXPECT_CALL的运行顺序。参数 *sequences* 可以任意多个 *Sequence* 类型的变量。如果想设置所有调用的顺序，可以直接在所有EXPECT_CALL之前声明一个 *InSequence* 类型的变量， 这样无需调用 *.InSequence* ，后面所有的函数都将按照 *EXPECT_CALL* 调用的顺序执行。但是如果我们只希望部分函数按照顺序执行而不关心其他部分的顺序呢，可以定义多个 *Sequence* 类型对象，所有调用 *.InSequence* 的 *EXPECT_CALL* 都将按照声明的顺序调用，上代码。
	```cpp
	// 设置全部顺序
	using ::testing::_;
	using ::testing::InSequence;
	{
	    InSequence s;
	    EXPECT_CALL(foo, DoThis(5));
	    EXPECT_CALL(bar, DoThat(_))
	        .Times(2);
	    EXPECT_CALL(foo, DoThis(6));
	}
	```
	```text
	调用顺序：foo.DoThis(5) -> bar.DoThat(_) -> bar.DoThat(_) -> foo.DoThis(6)
	```
	```cpp
	// 设置部分顺序
	using ::testing::Sequence;
	...
	  Sequence s1, s2;
	
	  EXPECT_CALL(foo, A())
	      .InSequence(s1, s2);
	  EXPECT_CALL(bar, B())
	      .InSequence(s1);
	  EXPECT_CALL(bar, C())
	      .InSequence(s2);
	  EXPECT_CALL(foo, D())
	      .InSequence(s2);
	```
	```text
	调用顺序：
	       +---> B
	       |
	  A ---|
	       |
	       +---> C ---> D
	```
 	- .After(expectations)
 	这个函数用来指定一个函数在某个或多个函数之后调用。参数 `expectations` 可以是最多五个 `Expectation` 类型变量( `EXPECT_CALL` 的返回值)，也可以是 `ExpectationSet` 类型变量( `Expectation` 的集合)。
 	```cpp
 	using ::testing::Expectation;
	...
	Expectation init_x = EXPECT_CALL(my_mock, InitX());
	Expectation init_y = EXPECT_CALL(my_mock, InitY());
	EXPECT_CALL(my_mock, Describe())
	    .After(init_x, init_y);
 	```
 	```text
 	my_mock.Describe() 在 my_mock.InitX() 和 my_mock.InitY() 都调用而完成后再调用。
 	```
 	```cpp
 	using ::testing::ExpectationSet;
	...
	ExpectationSet all_inits;
	// Collect all expectations of InitElement() calls
	for (int i = 0; i < element_count; i++) {
	  all_inits += EXPECT_CALL(my_mock, InitElement(i));
	}
	EXPECT_CALL(my_mock, Describe())
	    .After(all_inits);  // Expect Describe() call after all InitElement() calls
 	```
 	```text
 	my_mock.Describe() 在最后调用，这种使用方法在需要指定大量重复函数时很有用。
 	```
 	- .WillOnce(action)
 	这个函数设置被调用一次时的动作，只进行一次，不能重复。参数 `action` 表示动作。下面列举一些常用的动作，详细信息可以查阅文档 *googletest/doc/reference/actions.md* 。与`.Times(n)`一起使用时，调用`.WillOnce(action)`的次数应该与n相等，如果不等应该在后面调用`.WillRepeatedly(action)`，否则测试会失败。

	|||Action|说明|
	  |:--:|:--:|
	  |`::testing::Return`|返回空|
	  |`::testing::Return(value)`|返回value|
	  |`::testing::Invoke(f)`|f是一个函数，返回类型和参数类型与被模拟函数相同，可以是全局静态函数或累的成员|
	  |...|...|
	  - .WillRepeatedly(action)
	  这个函数可以认为是`.WillOnce(action)`的可重复版本。重复次数与`Times(n)`和实际调用次数有关。
	  - .RetiresOnSaturation()
	  这个函数的意义是：当被模拟的函数的调用次数达到指定上线时，这个预期的模拟将不再处于活跃状态。在下面的示例中，`m_mock.SetNumber(7)` 前两次的调用满足预期2，此时预期2将不再处于活跃状态，从第三次开始只会满足预期1。
	  ```cpp
	  using ::testing::_;
	using ::testing::AnyNumber;
	...
	EXPECT_CALL(my_mock, SetNumber(_))  // Expectation 1
	    .Times(AnyNumber());
	EXPECT_CALL(my_mock, SetNumber(7))  // Expectation 2
	    .Times(2)
	    .RetiresOnSaturation();
	  ```
# 使用心得
- gtest在使用时应该是 ==*独立于*== 被测项目的，最好是新建一个控制台应用专门编写测试。
- 如果一个模块(函数)的输入、输出以及功能都很清晰，那么这个模块的测试流程会比较简单，相反如果一个模块实现了多个功能，编写测试时就要考虑多种功能之间的影响和组合，比如前一个功能的运行结果是否会影响下一个功能，这时测试会变得很复杂，所以尽量保证我们的代码 ==*每个模块只完成一个功能*==，便于测试也便于维护。
- 一个好的测试应该是可重用的，这样不管是初期开发还是后期维护都能享受到测试带来的好处，因此在设计程序的时候尽量让业务模块(以后有可能会有变动的模块) ==*接口化*==，这样即便未来业务模块真的有变动，我们也能把修改代码的代价降到最小(可能只需要修改接口的实现或替换调用的接口就可以了)。
