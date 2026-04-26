# آموزش C++ برای فهمیدن کدهای Drogon

این آموزش فقط مفاهیمی رو پوشش می‌ده که برای خوندن و نوشتن کد با فریمورک Drogon نیاز داری. ترتیب از ساده به پیچیده‌ست و هر بخش با مثال از خود Drogon همراهه.

---

## ۱. ساختار کلی یه فایل C++

### چرا دو فایل؟

در C++ معمولاً هر کلاس در دو فایل جداگانه نوشته می‌شه:

| نوع | پسوند | محتوا | تشبیه |
|-----|-------|--------|-------|
| Header | `.h` | تعریف و امضای کلاس | فهرست محتویات |
| Source | `.cc` یا `.cpp` | پیاده‌سازی واقعی | خود محتویات |

فایل header مثل فهرست یه کتابه — بهت می‌گه چی هست ولی جزئیات نداره. فایل source مثل خود صفحات کتابه.

این جداسازی چند مزیت داره:
- وقتی یه فایل `.cc` دیگه می‌خواد از کلاس تو استفاده کنه، فقط header رو include می‌کنه، نه همه کد رو
- کامپایل سریع‌تره چون فقط فایل‌های تغییرکرده دوباره کامپایل می‌شن
- رابط عمومی کلاس (چیزی که دیگران می‌بینن) از پیاده‌سازی داخلی جداست

```cpp
// TestCtrl.h — فقط اعلام می‌کنیم که چه چیزی وجود داره
#pragma once
#include <drogon/HttpSimpleController.h>

class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
{
public:
    virtual void asyncHandleHttpRequest(...) override;
};
```

```cpp
// TestCtrl.cc — اینجاست که کد واقعی می‌نویسیم
#include "TestCtrl.h"

void TestCtrl::asyncHandleHttpRequest(...)
{
    // کد اصلی اینجاست
}
```

### `#pragma once`

این دستور به کامپایلر می‌گه این فایل header رو فقط یه بار پردازش کن، حتی اگه چندین جای مختلف include شده باشه. بدونش ممکنه تعریف یه کلاس دو بار دیده بشه و خطا بگیری.

### `#include`

دو شکل داره:

```cpp
#include <drogon/HttpSimpleController.h>   // با < > : فایل‌های کتابخونه‌های نصب‌شده
#include "TestCtrl.h"                       // با " " : فایل‌های خود پروژه
```

تفاوت فقط در جایی‌که کامپایلر دنبال فایل می‌گرده‌ست. `< >` توی پوشه‌های سیستمی می‌گرده، `" "` اول توی پوشه فعلی.

---

## ۲. انواع داده پایه

قبل از هر چیز، باید با انواع داده پایه آشنا بشی:

```cpp
int age = 25;           // عدد صحیح (مثبت و منفی)
unsigned int count = 0; // عدد صحیح فقط مثبت
double price = 19.99;   // عدد اعشاری
bool isActive = true;   // درست/غلط (true یا false)
char letter = 'A';      // یه کاراکتر تکی
std::string name = "Pooria";  // رشته متن
```

### تعریف متغیر

```cpp
int x;          // تعریف (مقدار اولیه نداره — خطرناکه)
int x = 10;     // تعریف با مقداردهی اولیه
int x{10};      // روش مدرن‌تر مقداردهی (C++11 به بعد)
auto x = 10;    // نوع رو خودکار تشخیص بده (اینجا int می‌شه)
```

### `std::string` در عمل

```cpp
std::string path = "/api/users";

path.length();          // طول رشته: 11
path.substr(5);         // از کاراکتر پنجم به بعد: "users"
path.find("/api");      // جای اولین اتفاق: 0
path + "/active";       // ترکیب: "/api/users/active"
path == "/api/users";   // مقایسه: true
```

در Drogon خیلی از متدها `std::string` برمی‌گردونن:
```cpp
std::string path = req->getPath();
std::string body = req->getBody();
```

---

## ۳. Namespace — فضای نام

### مشکلی که Namespace حل می‌کنه

فرض کن دو کتابخونه مختلف هر دو یه تابع به اسم `log` دارن. C++ نمی‌دونه کدوم رو صدا بزنه. Namespace این مشکل رو حل می‌کنه:

```cpp
drogon::app()    // تابع app از کتابخونه drogon
mylib::app()     // تابع app از کتابخونه mylib — کاملاً جداست
```

### `::` چطور کار می‌کنه

نماد `::` یعنی "داخل این". مثل مسیر پوشه‌بندی‌ه:

```cpp
drogon::HttpAppFramework    // کلاس HttpAppFramework داخل namespace دروگون
std::string                 // کلاس string داخل namespace استاندارد
```

### `using namespace`

```cpp
using namespace drogon;
```

این دستور می‌گه "هر وقت اسمی پیدا نشد، توی drogon هم دنبالش بگرد". نتیجه‌اش اینه که دیگه لازم نیست `drogon::` بنویسی:

```cpp
// بدون using namespace:
drogon::app().addListener("0.0.0.0", 80);
drogon::app().run();

// با using namespace drogon:
app().addListener("0.0.0.0", 80);
app().run();
```

> **نکته:** در فایل‌های `.h` از `using namespace` استفاده نکن. چون هر کسی که این header رو include کنه ناخواسته اون namespace رو هم می‌گیره. فقط در فایل‌های `.cc` استفاده کن.

---

## ۴. تابع (Function)

### ساختار یه تابع

```cpp
نوع_خروجی  اسم_تابع(پارامترها)
{
    // بدنه تابع
    return مقدار;  // اگه خروجی void نباشه
}
```

مثال واقعی:

```cpp
// تابعی که یه عدد می‌گیره و دو برابرش رو برمی‌گردونه
int double_it(int x)
{
    return x * 2;
}

// تابعی که هیچی برنمی‌گردونه (void)
void printMessage(std::string msg)
{
    // فقط کاری انجام می‌ده
}
```

### متدهای کلاس

وقتی یه تابع داخل یه کلاس تعریف می‌شه، بهش می‌گیم متد. برای تعریف متد خارج از کلاس (توی فایل `.cc`) باید اسم کلاس رو هم بنویسی:

```cpp
// اینجا داریم می‌گیم: متد asyncHandleHttpRequest متعلق به کلاس TestCtrl
void TestCtrl::asyncHandleHttpRequest(
    const HttpRequestPtr &req,
    std::function<void(const HttpResponsePtr &)> &&callback)
{
    // بدنه
}
```

### تابع‌های Static

تابع `static` داخل یه کلاس بدون نیاز به ساختن instance کلاس قابل صدا زدنه:

```cpp
// newHttpResponse یه متد static هست
// یعنی بدون ساختن HttpResponse می‌تونی صداش بزنی
auto resp = HttpResponse::newHttpResponse();
//                       ↑ مستقیم از کلاس صداش زدیم، نه از یه object
```

---

## ۵. Class و Inheritance — کلاس و وراثت

### کلاس چیست؟

کلاس یه قالب (blueprint) برای ساختن object هاست. مثل نقشه ساختمان — نقشه خودش ساختمان نیست، ولی از روش می‌شه ساختمان ساخت.

```cpp
class Car {
public:
    std::string brand;   // ویژگی (attribute/member variable)
    int speed;

    void accelerate() {  // رفتار (method/member function)
        speed += 10;
    }
};

// ساختن یه object از کلاس Car:
Car myCar;
myCar.brand = "Toyota";
myCar.accelerate();
```

### سطوح دسترسی: `public` و `private`

```cpp
class TestCtrl
{
public:
    // اینجا چیزایی که کد خارجی می‌تونه ببینه و استفاده کنه
    void asyncHandleHttpRequest(...);

private:
    // اینجا چیزایی که فقط داخل خود این کلاس قابل استفاده‌ست
    int requestCount = 0;
    void helperFunction();
};
```

اگه `public` یا `private` ننویسی، پیش‌فرض `private`ه.

### Inheritance — وراثت

وراثت یعنی یه کلاس همه ویژگی‌ها و رفتارهای یه کلاس دیگه رو بگیره و چیزی بهش اضافه کنه یا تغییرش بده.

```cpp
class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
//    فرزند      ↑             پدر
```

**`TestCtrl`** فرزنده — کلاس جدیدیه که داری می‌سازی  
**`HttpSimpleController`** پدره — کلاسی که Drogon قبلاً نوشته  
**`public`** یعنی همه چیز `public` توی پدر، توی فرزند هم `public` می‌مونه

به زبان ساده: Drogon یه کلاس پایه نوشته که خیلی کارها می‌کنه (اتصال به شبکه، پارس کردن HTTP، مدیریت thread ها و...). تو فقط از اون ارث می‌بری و می‌گی "وقتی request اومد، این کار رو بکن".

### چرا وراثت؟

بدون وراثت، برای هر controller باید از صفر همه چیز می‌نوشتی. با وراثت، Drogon کارهای سخت رو کرده و تو فقط منطق اپلیکیشنت رو می‌نویسی.

---

## ۶. Pointer و Reference — اشاره‌گر و ارجاع

این یکی از مهم‌ترین و در عین حال گیج‌کننده‌ترین مفاهیم C++ هست. با مثال توضیح می‌دم.

### حافظه چطور کار می‌کنه

تصور کن RAM مثل یه خیابان بلنده که هر خونه یه آدرس داره. وقتی یه متغیر می‌سازی، یه خونه توی این خیابان رزرو می‌شه:

```
آدرس: 1000    مقدار: 42      ← int x = 42;
آدرس: 1004    مقدار: 3.14    ← double y = 3.14;
```

### Pointer — `*`

Pointer یه متغیریه که به جای ذخیره یه مقدار، **آدرس** یه خونه دیگه رو ذخیره می‌کنه:

```cpp
int x = 42;
int* p = &x;   // p آدرس x رو داره
               // & اینجا یعنی "آدرس"

// p خودش ← آدرس 1000 (یعنی اینجاست که x نشسته)
// *p      ← مقدار اون خونه = 42  (با * به مقدار دسترسی داری)
```

در Drogon خیلی از چیزها به صورت pointer رد و بدل می‌شن:
```cpp
HttpRequestPtr req
// این یه pointer به یه HttpRequest هست
// وقتی Drogon یه request می‌گیره، یه HttpRequest می‌سازه و آدرسشو بهت می‌ده
```

برای دسترسی به اعضای یه object از طریق pointer از `->` استفاده می‌کنی:
```cpp
req->getPath()     // مسیر URL رو بده
req->getMethod()   // متد HTTP رو بده (GET/POST/...)
req->getBody()     // بدنه request رو بده
req->getHeader("Content-Type")  // یه هدر خاص رو بده
```

این معادله با `.` هست، با یه تفاوت:
```cpp
(*req).getPath()   // اول محتوا رو بگیر، بعد متد رو صدا بزن
req->getPath()     // همینه، ولی کوتاه‌تر
```

### Reference — `&`

Reference مثل یه اسم مستعاره — همون متغیر اصلیه، فقط یه اسم دیگه داره:

```cpp
int x = 42;
int& ref = x;   // ref یه اسم دیگه برای x هست

ref = 100;      // در واقع x رو تغییر دادیم
// حالا x هم 100 هست
```

**چرا Reference در پارامترهای تابع مهمه؟**

وقتی یه متغیر به تابع می‌دی، معمولاً یه کپی ساخته می‌شه:

```cpp
void doSomething(HttpRequest req)  // ← یه کپی کامل از request ساخته می‌شه
                                   //   اگه request بزرگ باشه، کند و اتلاف حافظه‌ست
```

با Reference، کپی ساخته نمی‌شه — همون object اصلی رد می‌شه:
```cpp
void doSomething(HttpRequest& req)  // ← هیچ کپی‌ای ساخته نمی‌شه، سریع‌تره
```

### `const` — تغییرناپذیر

`const` یعنی "این مقدار نباید تغییر کنه". مثل قفل کردن یه متغیر:

```cpp
const int MAX = 100;    // MAX نمی‌تونه عوض بشه
MAX = 200;              // خطای کامپایل!
```

در پارامترهای Drogon معمولاً می‌بینی:
```cpp
const HttpRequestPtr &req
//  ↑ نمی‌تونی req رو تغییر بدی — فقط می‌تونی بخونیش
```

این یعنی: این reference رو بگیر (کپی نساز)، ولی اجازه نداری مقداری که بهش اشاره می‌کنه رو عوض کنی.

### خلاصه `*` و `&`

```cpp
int x = 42;
int* p = &x;    // & اینجا: "آدرس x رو بده" — می‌شه pointer
int& r = x;     // & اینجا: "r یه اسم دیگه برای x هست" — می‌شه reference

void f(int& r)  // & اینجا: "بدون کپی بده" — پارامتر reference
```

---

## ۷. `auto` — تشخیص خودکار نوع

در C++ هر متغیر یه نوع مشخص داره. گاهی نوع خیلی طولانیه:

```cpp
// قدیمی:
std::shared_ptr<HttpResponse> resp = HttpResponse::newHttpResponse();

// با auto:
auto resp = HttpResponse::newHttpResponse();
// کامپایلر می‌فهمه که resp از نوع std::shared_ptr<HttpResponse> هست
```

`auto` نوع رو از مقدار سمت راست `=` استنتاج می‌کنه. اگه مقدار سمت راست `int` باشه، متغیر `int` می‌شه:

```cpp
auto x = 42;          // int
auto y = 3.14;        // double
auto z = true;        // bool
auto s = std::string("hello");  // std::string
auto resp = HttpResponse::newHttpResponse();  // shared_ptr<HttpResponse>
```

`auto` مقدار رو تغییر نمی‌ده — فقط نوشتن رو کوتاه‌تر می‌کنه.

---

## ۸. `std::function` — تابع به عنوان مقدار

### Callback چیست؟

Callback یه الگوی برنامه‌نویسیه که می‌گی: "این کار رو بکن، و وقتی تموم شد این تابع رو صدا بزن."

مثال روزمره: وقتی به یه رستوران زنگ می‌زنی و می‌گی "پیتزا آماده شد زنگ بزن" — اون callback هست.

در Drogon، به خاطر async بودن، request رو پردازش می‌کنی و جواب رو از طریق callback می‌فرستی:

```cpp
void TestCtrl::asyncHandleHttpRequest(
    const HttpRequestPtr &req,
    std::function<void(const HttpResponsePtr &)> &&callback)
//  ↑ این یه تابع callback هست که باید بهش response بدی
{
    auto resp = HttpResponse::newHttpResponse();
    resp->setBody("Hello!");
    callback(resp);  // ← اینجا callback رو صدا می‌زنی و response رو می‌فرستی
}
```

### شکستن `std::function<void(const HttpResponsePtr &)>`

```
std::function < void ( const HttpResponsePtr & ) >
     ↑                  ↑        ↑
 نوع تابع         نوع خروجی   پارامتر ورودی
```

- `std::function<...>` — یه container هست که می‌تونه هر تابعی رو نگه داره
- `void` — خروجی نداره
- `const HttpResponsePtr &` — یه response به عنوان ورودی می‌گیره

### `&&` — Move Reference

```cpp
std::function<...> &&callback
//                 ↑↑
//           دو تا & یعنی move reference
```

Move یعنی "این object رو منتقل کن، کپی نکن". برای callback این بهینه‌تره چون callback فقط یه بار استفاده می‌شه. فعلاً فقط بدون که باید `&&` بنویسی — بعداً می‌فهمیش.

---

## ۹. `virtual` و `override` — بازنویسی توابع

### مشکل بدون `virtual`

فرض کن Drogon یه کلاس پایه نوشته و توی اون تابع `handle` داره:

```cpp
// کلاس پدر توی Drogon:
class HttpSimpleController {
public:
    void asyncHandleHttpRequest(...) {
        // کد پیش‌فرض — هیچی نمی‌کنه
    }
};
```

اگه تو یه فرزند بسازی و همین تابع رو بنویسی، بدون `virtual` ممکنه Drogon همچنان تابع پدر رو صدا بزنه نه تابع تو رو!

### `virtual` چطور کمک می‌کنه

با `virtual`، وقتی Drogon این تابع رو صدا می‌زنه، C++ می‌فهمه که باید پیاده‌سازی فرزند رو صدا بزنه:

```cpp
// کلاس پدر توی Drogon:
virtual void asyncHandleHttpRequest(...);
//  ↑ اگه فرزند این تابع رو بازنویسی کرده، نسخه فرزند رو صدا بزن
```

### `override` برای اطمینان

`override` به کامپایلر می‌گه "من دارم یه تابع `virtual` از پدر رو بازنویسی می‌کنم". اگه اشتباه کنی (اسم غلط، پارامتر غلط)، کامپایلر خطا می‌ده:

```cpp
// در TestCtrl.h:
virtual void asyncHandleHttpRequest(
    const HttpRequestPtr &req,
    std::function<void(const HttpResponsePtr &)> &&callback
) override;
//  ↑ کامپایلر چک می‌کنه که واقعاً داری یه virtual از پدر رو override می‌کنی
```

---

## ۱۰. Template — قالب عمومی

### Template چیست؟

Template یه روش برای نوشتن کدیه که با "هر نوعی" کار کنه. مثلاً تابعی که هم با `int` و هم با `double` کار کنه:

```cpp
// بدون template — باید دو تابع بنویسی:
int max_int(int a, int b) { return a > b ? a : b; }
double max_double(double a, double b) { return a > b ? a : b; }

// با template — یه تابع برای همه:
template<typename T>
T max_value(T a, T b) { return a > b ? a : b; }

max_value(3, 5);      // T = int
max_value(3.2, 5.7);  // T = double
```

### Template در Drogon — CRTP

در Drogon می‌بینی:
```cpp
class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
//                                                   ↑ اسم خودت رو بهش می‌دی
```

این یه تکنیک به اسم **CRTP (Curiously Recurring Template Pattern)** هست. دلیلش اینه که Drogon باید بدونه چه کلاسی داره ازش ارث می‌بره تا بتونه instance درست بسازه.

**نکته عملی:** همیشه اسم کلاست رو داخل `<>` بنویس. تغییر دیگه‌ای لازم نیست.

---

## ۱۱. Smart Pointer — اشاره‌گر هوشمند

### مشکل Pointer معمولی

در C++ قدیمی، وقتی با `new` حافظه می‌گرفتی، باید خودت با `delete` آزادش می‌کردی:

```cpp
HttpRequest* req = new HttpRequest();  // حافظه گرفتیم
// ... کارها ...
delete req;  // باید فراموش نکنی! وگرنه memory leak داری
```

اگه فراموش می‌کردی `delete` بنویسی، یا exception می‌خورد و به `delete` نمی‌رسیدی، حافظه هدر می‌رفت.

### Smart Pointer — راه‌حل

Smart pointer خودش وقتی دیگه لازم نیست، حافظه رو آزاد می‌کنه:

```cpp
std::shared_ptr<HttpRequest> req = std::make_shared<HttpRequest>();
// وقتی req از scope خارج بشه یا کسی دیگه ازش استفاده نکنه، خودش حذف می‌شه
```

**`shared_ptr`** یه شمارنده داره — می‌دونه چند نفر دارن از این object استفاده می‌کنن. وقتی به صفر رسید، حذف می‌شه.

### در Drogon

Drogon از smart pointer استفاده می‌کنه و اسم‌های کوتاه‌تری براشون گذاشته:

```cpp
HttpRequestPtr   // = std::shared_ptr<HttpRequest>
HttpResponsePtr  // = std::shared_ptr<HttpResponse>
```

این‌ها با `using` تعریف شدن:
```cpp
using HttpRequestPtr = std::shared_ptr<HttpRequest>;
```

استفاده ازشون مثل pointer معمولیه (با `->`)، ولی نگران حذف کردنشون نیستی.

---

## ۱۲. Enum — مقادیر ثابت با اسم

### Enum چیست؟

وقتی یه مجموعه مقدار ثابت داری که به هم مرتبطن، از enum استفاده می‌کنی:

```cpp
enum Color { Red, Green, Blue };
Color myColor = Red;
```

به جای اینکه بنویسی `0` یا `1` یا `2`، اسم می‌دی — خواناتره و کمتر اشتباه می‌کنی.

### در Drogon

```cpp
// کدهای HTTP به صورت enum تعریف شدن:
resp->setStatusCode(k200OK);    // 200 - موفق
resp->setStatusCode(k404NotFound);   // 404 - پیدا نشد
resp->setStatusCode(k500InternalServerError);  // 500 - خطای سرور

// نوع محتوا هم enum هست:
resp->setContentTypeCode(CT_TEXT_HTML);    // text/html
resp->setContentTypeCode(CT_APPLICATION_JSON);  // application/json
```

این عدد ۲۰۰ نیست — یه اسم معنادار هست که اتفاقاً مقدارش ۲۰۰ هست.

---

## ۱۳. Macro — ماکرو

### Macro چیست؟

Macro یه متن‌جایگزینی ساده‌ست. قبل از کامپایل، preprocessor همه جاهایی که macro استفاده شده رو با متن تعریف‌شده جایگزین می‌کنه:

```cpp
#define MAX_SIZE 100

int arr[MAX_SIZE];  // ← قبل از کامپایل می‌شه: int arr[100];
```

### Macro در Drogon

Drogon از macro برای syntax راحت‌تر استفاده می‌کنه:

```cpp
PATH_LIST_BEGIN
    PATH_ADD("/", Get, Post);
    PATH_ADD("/test", Get);
PATH_LIST_END
```

این macro ها در واقع کد C++ پیچیده‌ای رو generate می‌کنن که route رو ثبت می‌کنه. نیازی نیست بدونی داخلشون چیه — فقط نحوه استفاده رو بدون.

> **نکته:** Macro ها معمولاً با حروف بزرگ نوشته می‌شن تا از متغیرها و توابع عادی متمایز باشن.

---

## ۱۴. Scope — محدوده متغیر

هر متغیر فقط در "محدوده" (scope) خودش زنده‌ست. Scope با `{` و `}` مشخص می‌شه:

```cpp
void myFunction() {
    int x = 10;          // x اینجا به وجود میاد

    if (x > 5) {
        int y = 20;      // y اینجا به وجود میاد
        int z = x + y;   // هر دو در دسترسن
    }
    // y و z اینجا دیگه وجود ندارن — از scope خارج شدن

    // x هنوز در دسترسه
}
// x اینجا هم دیگه وجود نداره
```

در Drogon، وقتی `callback` رو صدا می‌زنی باید مطمئن بشی همه چیزی که callback نیاز داره هنوز زنده‌ست.

---

## ۱۵. Lambda — تابع ناشناس

Lambda یه تابع بدون اسمه که می‌تونی مستقیم تعریفش کنی:

```cpp
// تابع معمولی:
void printHello() {
    std::cout << "Hello!";
}

// lambda معادلش:
auto printHello = []() {
    std::cout << "Hello!";
};
```

### Lambda در Drogon

در Drogon خیلی وقتا lambda می‌بینی، مخصوصاً در database queries:

```cpp
// مثال از Drogon با database:
db->execSqlAsync(
    "SELECT * FROM users",
    [](const drogon::orm::Result &r) {
        // این lambda وقتی query تموم شد صدا زده می‌شه
        for (auto row : r) {
            auto name = row["name"].as<std::string>();
        }
    },
    [](const drogon::orm::DrogonDbException &e) {
        // این lambda اگه خطا بود صدا زده می‌شه
    }
);
```

### ساختار Lambda

```cpp
[capture](parameters) -> return_type {
    body
}
```

- **`[]`** — capture list: چه متغیرهایی از بیرون داخل lambda قابل استفاده باشن
- **`()`** — پارامترها مثل تابع معمولی
- **بدنه** — کد تابع

```cpp
int multiplier = 3;

// با [multiplier] می‌گیم می‌خوایم multiplier رو از بیرون استفاده کنیم
auto triple = [multiplier](int x) {
    return x * multiplier;
};

triple(5);  // 15
```

---

## ۱۶. کد کامل Drogon با توضیح خط به خط

### فایل header — TestCtrl.h

```cpp
#pragma once
// این فایل فقط یه بار include بشه،
// حتی اگه چند جا include شده باشه

#include <drogon/HttpSimpleController.h>
// کلاس پایه HttpSimpleController رو بیار که بتونیم ازش ارث ببریم

using namespace drogon;
// دیگه لازم نیست drogon:: قبل هر چیزی بنویسیم

class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
// TestCtrl کلاس جدید ماست
// از HttpSimpleController ارث می‌بره (یعنی همه کارهای Drogon رو می‌کنه)
// <TestCtrl> پاس دادن خودمون به template هست (الگوی CRTP)
{
public:
// همه چیزی که بعد از اینجاست از بیرون قابل دسترسه

    virtual void asyncHandleHttpRequest(
    // virtual: این تابع می‌تونه override بشه
    // async: این تابع به صورت async کار می‌کنه (بلاک نمی‌شه)

        const HttpRequestPtr &req,
        // const: نمی‌تونی req رو تغییر بدی
        // HttpRequestPtr: smart pointer به HttpRequest
        // &: بدون کپی، reference پاس می‌شه

        std::function<void(const HttpResponsePtr &)> &&callback
        // std::function: یه تابع که می‌شه ذخیره‌ش کرد
        // void(const HttpResponsePtr &): این تابع یه response می‌گیره و چیزی برنمی‌گردونه
        // &&: move reference — کپی نمی‌شه

    ) override;
    // override: داریم تابع کلاس پدر رو بازنویسی می‌کنیم
    // کامپایلر چک می‌کنه که این تابع واقعاً توی پدر وجود داره

    PATH_LIST_BEGIN
    // ماکرو Drogon: شروع تعریف مسیرها

        PATH_ADD("/", Get, Post);
        // مسیر "/" با متدهای GET و POST پشتیبانی می‌شه

        PATH_ADD("/test", Get);
        // مسیر "/test" فقط GET

    PATH_LIST_END
    // ماکرو Drogon: پایان تعریف مسیرها
};
```

### فایل source — TestCtrl.cc

```cpp
#include "TestCtrl.h"
// header خودمون رو include می‌کنیم
// " " یعنی توی پوشه خودمون دنبالش بگرد

void TestCtrl::asyncHandleHttpRequest(
// TestCtrl:: یعنی این تابع متعلق به کلاس TestCtrl هست
    const HttpRequestPtr &req,
    std::function<void(const HttpResponsePtr &)> &&callback)
{
    auto resp = HttpResponse::newHttpResponse();
    // newHttpResponse یه متد static هست — بدون ساختن object صداش می‌زنیم
    // auto: کامپایلر نوع رو می‌فهمه (shared_ptr<HttpResponse>)

    resp->setStatusCode(k200OK);
    // -> چون resp یه pointer هست
    // k200OK یه enum هست با مقدار 200

    resp->setContentTypeCode(CT_TEXT_HTML);
    // CT_TEXT_HTML یه enum هست برای "text/html"

    resp->setBody("Hello World!");
    // متن پاسخ رو تنظیم می‌کنیم

    callback(resp);
    // callback رو صدا می‌زنیم و response رو بهش می‌دیم
    // Drogon این response رو به client می‌فرسته
}
```

---

## ۱۷. چطور با JSON کار کنیم (در Drogon)

خیلی از API ها با JSON کار می‌کنن. Drogon یه library به اسم `Json::Value` داره:

```cpp
#include <json/json.h>

void MyCtrl::asyncHandleHttpRequest(
    const HttpRequestPtr &req,
    std::function<void(const HttpResponsePtr &)> &&callback)
{
    // ساختن یه JSON response:
    Json::Value result;
    result["status"] = "ok";
    result["message"] = "Hello!";
    result["count"] = 42;

    // آرایه JSON:
    Json::Value arr(Json::arrayValue);
    arr.append("item1");
    arr.append("item2");
    result["items"] = arr;

    // فرستادن JSON:
    auto resp = HttpResponse::newHttpJsonResponse(result);
    callback(resp);
}
```

### خوندن JSON از request

```cpp
auto json = req->getJsonObject();  // دریافت body به عنوان JSON

if (json) {  // چک کن که JSON معتبر بود
    std::string name = (*json)["name"].asString();
    int age = (*json)["age"].asInt();
    bool active = (*json)["active"].asBool();
}
```

---

## ۱۸. خلاصه نمادها و کلمات کلیدی

| نماد/کلمه | معنا | مثال |
|-----------|------|-------|
| `::` | داخل این namespace یا کلاس | `drogon::app()` |
| `->` | دسترسی به عضو از طریق pointer | `req->getPath()` |
| `.` | دسترسی به عضو مستقیم | `myObj.name` |
| `&` در تعریف | reference — نه کپی | `const Req &req` |
| `&` در expression | آدرس متغیر | `&x` |
| `*` در تعریف | pointer — نگه‌داری آدرس | `int* p` |
| `*` در expression | محتوای pointer | `*p` |
| `const` | تغییرناپذیر | `const int x = 5` |
| `auto` | نوع خودکار | `auto x = 5` |
| `virtual` | قابل override | `virtual void f()` |
| `override` | دارم override می‌کنم | `void f() override` |
| `public:` | از بیرون قابل دسترس | |
| `private:` | فقط داخل کلاس | |
| `<T>` | template با نوع T | `shared_ptr<int>` |
| `[](){}` | lambda (تابع ناشناس) | `[x](int y){ return x+y; }` |
| `&&` | move reference | `string &&s` |
| `using` | اسم مستعار برای نوع | `using Ptr = shared_ptr<X>` |

---

## ۱۹. چیزایی که لازم نیست یاد بگیری (هنوز)

- **Memory management دستی** (`new` / `delete`) — Drogon از smart pointer استفاده می‌کنه
- **Move semantics عمیق** — فقط بدون `&&` برای callback وجود داره
- **Template metaprogramming** — فقط بدون که `<TestCtrl>` رو باید بنویسی
- **Concurrency و thread دستی** — Drogon خودش مدیریت می‌کنه
- **Operator overloading** — در کدهای Drogon نمی‌نویسی
- **Raw arrays** (`int arr[10]`) — از `std::vector` استفاده می‌شه
- **C-style strings** (`char*`) — از `std::string` استفاده می‌شه

Drogon یه API تمیز و مدرن داره که خیلی از پیچیدگی‌های C++ رو پشتش مخفی کرده. اگه این فایل رو بفهمی، می‌تونی کدهای Drogon رو بخونی و بنویسی.
