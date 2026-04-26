# آموزش C++ برای فهمیدن کدهای Drogon

این آموزش فقط مفاهیمی رو پوشش می‌ده که برای خوندن و نوشتن کد با فریمورک Drogon نیاز داری. ترتیب از ساده به پیچیده‌ست و هر بخش با مثال از خود Drogon همراهه.

---

## ۱. ساختار کلی یه فایل C++

هر فایل C++ از دو نوع فایل تشکیل می‌شه:

| نوع | پسوند | محتوا |
|-----|-------|--------|
| Header | `.h` | تعریف کلاس و توابع (مثل امضا) |
| Source | `.cc` یا `.cpp` | پیاده‌سازی (مثل بدنه) |

```cpp
// TestCtrl.h — فقط تعریف
#pragma once
#include <drogon/HttpSimpleController.h>

class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
{
public:
    virtual void asyncHandleHttpRequest(...) override;
};
```

```cpp
// TestCtrl.cc — پیاده‌سازی
#include "TestCtrl.h"

void TestCtrl::asyncHandleHttpRequest(...)
{
    // کد اصلی اینجاست
}
```

**`#pragma once`** یعنی این فایل فقط یه بار include بشه.  
**`#include`** یعنی محتوای اون فایل رو اینجا بیار.

---

## ۲. Namespace — فضای نام

Namespace از تداخل اسم‌ها جلوگیری می‌کنه. مثل پوشه‌بندی برای اسم‌هاست.

```cpp
drogon::app().run();
//  ↑
// یعنی تابع app() که داخل namespace به اسم drogon تعریف شده
```

وقتی می‌نویسی:
```cpp
using namespace drogon;
```
دیگه لازم نیست `drogon::` قبل همه چیز بنویسی:
```cpp
app().run();  // همون معناست
```

---

## ۳. Class و Inheritance — کلاس و وراثت

### کلاس چیست؟

کلاس یه قالبه. مثل یه دستورالعمل برای ساختن یه چیز.

```cpp
class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
//    ↑ اسم کلاس     ↑ از این کلاس ارث می‌بره (ویژگی‌هاش رو داره)
```

**`public`** در اینجا یعنی همه ویژگی‌های کلاس پدر در دسترس هستن.

### `public:` داخل کلاس

```cpp
class TestCtrl
{
public:          // ← همه چیز بعد از این، از بیرون هم قابل دسترسه
    void doSomething();

private:         // ← فقط داخل خود کلاس قابل دسترسه
    int counter;
};
```

---

## ۴. Pointer و Reference — اشاره‌گر و ارجاع

این یکی از مهم‌ترین مفاهیمه چون در Drogon خیلی می‌بینیش.

### Pointer — `*`

به جای ذخیره‌ی یه مقدار، **آدرس** اون رو ذخیره می‌کنه.

```cpp
HttpRequestPtr req
// ↑ این یه pointer به یه HttpRequest هست
// یعنی req آدرس یه request رو داره، نه خود request رو
```

وقتی pointer داری، برای دسترسی به ویژگی‌هاش از `->` استفاده می‌کنی:
```cpp
req->getPath()    // مسیر URL رو بده
req->getMethod()  // متد HTTP رو بده (GET/POST/...)
```

### Reference — `&`

Reference یه اسم دیگه برای همون متغیره. وقتی با `&` می‌فرستی، یه کپی جدید ساخته نمی‌شه:

```cpp
void test(const HttpRequestPtr &req)
//                              ↑ ref به req، نه کپی
```

### `const`

یعنی این مقدار تغییر نمی‌کنه:
```cpp
const HttpRequestPtr &req
// ↑ نمی‌تونی req رو تغییر بدی، فقط می‌تونی بخونیش
```

---

## ۵. `auto` — تشخیص خودکار نوع

C++ مدرن می‌تونه نوع متغیر رو خودش بفهمه:

```cpp
auto resp = HttpResponse::newHttpResponse();
// به جای اینکه بنویسی:
// std::shared_ptr<HttpResponse> resp = HttpResponse::newHttpResponse();
```

`auto` فقط یعنی "نوعش رو خودت بفهم". کدت رو کوتاه‌تر می‌کنه.

---

## ۶. `std::function` — تابع به عنوان مقدار

در Drogon، callback رو به صورت تابع می‌فرستی:

```cpp
std::function<void(const HttpResponsePtr &)> &&callback
```

بیا این رو بشکنیم:
- `std::function<...>` — یه تابع که می‌شه ذخیره‌ش کرد
- `void(const HttpResponsePtr &)` — این تابع هیچی برنمی‌گردونه و یه HttpResponse می‌گیره
- `&&callback` — با `&&` یعنی این تابع منتقل می‌شه (move)، نه کپی

استفاده ازش خیلی ساده‌ست:
```cpp
callback(resp);  // تابع رو صدا بزن و resp رو بهش بده
```

---

## ۷. `virtual` و `override`

### `virtual`

یعنی این تابع می‌تونه در کلاس فرزند دوباره تعریف بشه:

```cpp
// در کلاس پدر (HttpSimpleController):
virtual void asyncHandleHttpRequest(...);
```

### `override`

یعنی "من دارم همین تابع پدر رو بازنویسی می‌کنم":

```cpp
// در کلاس فرزند (TestCtrl):
virtual void asyncHandleHttpRequest(...) override;
```

اگه اشتباه تایپ کنی، کامپایلر خطا می‌ده. خیلی مفیده.

---

## ۸. Template — قالب عمومی

در Drogon می‌بینی:
```cpp
class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
//                                                   ↑ Template argument
```

Template یعنی "این کلاس با هر نوعی کار می‌کنه". اینجا Drogon از یه تکنیک به اسم **CRTP** استفاده می‌کنه — کلاس رو به خودش پاس می‌ده تا بتونه instance بسازه. نیازی نیست عمیقاً بدونی چطور کار می‌کنه، فقط بدون که باید اسم کلاست رو داخل `<>` بنویسی.

---

## ۹. Scope Resolution — `::`

دو نقطه پشت هم (`::`) یعنی "داخل این namespace یا کلاس":

```cpp
drogon::app()              // تابع app() داخل namespace drogon
HttpResponse::newHttpResponse()  // تابع static داخل کلاس HttpResponse
k200OK                     // مقدار enum (نیازی به :: نداره، داخل namespace دروگونه)
```

---

## ۱۰. Enum — مقادیر ثابت با اسم

به جای اینکه بنویسی عدد `200`، می‌نویسی:
```cpp
resp->setStatusCode(k200OK);
```

`k200OK` یه enum هست که مقدارش `200` هست. کد خواناتره.

---

## ۱۱. Smart Pointer — اشاره‌گر هوشمند

در Drogon همه جا می‌بینی:
```cpp
HttpRequestPtr
HttpResponsePtr
```

اینا در واقع `std::shared_ptr<HttpRequest>` و `std::shared_ptr<HttpResponse>` هستن.

**Smart pointer** حافظه رو خودش مدیریت می‌کنه — وقتی دیگه کسی ازش استفاده نمی‌کنه، خودش حذف می‌شه. نیازی نیست `delete` بنویسی.

---

## ۱۲. Macro — ماکرو

در Drogon ماکروهایی می‌بینی که با حروف بزرگ نوشته شدن:

```cpp
PATH_LIST_BEGIN
    PATH_ADD("/", Get, Post);
    PATH_ADD("/test", Get);
PATH_LIST_END
```

ماکرو در واقع یه shortcut هست که قبل از کامپایل، متن رو جایگزین می‌کنه. مثل copy-paste اتوماتیک. نیازی نیست بدونی داخلشون چیه — فقط از رابطشون استفاده کن.

---

## ۱۳. یه کد کامل Drogon با توضیح خط به خط

```cpp
#pragma once                          // این فایل فقط یه بار include بشه
#include <drogon/HttpSimpleController.h>  // هدر Drogon رو بیار

using namespace drogon;               // دیگه لازم نیست drogon:: بنویسیم

class TestCtrl                        // یه کلاس جدید به اسم TestCtrl
    : public drogon::HttpSimpleController<TestCtrl>  // از این کلاس ارث می‌بره
{
public:                               // همه چیز زیر اینجا از بیرون قابل دسترسه

    virtual void asyncHandleHttpRequest(
        const HttpRequestPtr &req,    // اشاره‌گر به request (تغییرناپذیر)
        std::function<void(const HttpResponsePtr &)> &&callback  // تابع callback
    ) override;                       // داریم تابع کلاس پدر رو override می‌کنیم

    PATH_LIST_BEGIN                   // شروع لیست مسیرها
        PATH_ADD("/", Get, Post);     // مسیر "/" با متدهای GET و POST
        PATH_ADD("/test", Get);       // مسیر "/test" فقط با GET
    PATH_LIST_END                     // پایان لیست مسیرها
};
```

```cpp
#include "TestCtrl.h"   // هدر همین کلاس رو include کن

void TestCtrl::asyncHandleHttpRequest(   // پیاده‌سازی تابع
    const HttpRequestPtr &req,
    std::function<void(const HttpResponsePtr &)> &&callback)
{
    auto resp = HttpResponse::newHttpResponse();  // یه response جدید بساز
    resp->setStatusCode(k200OK);                  // کد 200 (موفق)
    resp->setContentTypeCode(CT_TEXT_HTML);        // نوع محتوا: HTML
    resp->setBody("Hello World!");                 // متن پاسخ
    callback(resp);                                // پاسخ رو برگردون
}
```

---

## ۱۴. خلاصه سریع برای مرور

| نماد | معنا |
|------|------|
| `::` | داخل این namespace یا کلاس |
| `->` | دسترسی به عضو از طریق pointer |
| `.` | دسترسی به عضو مستقیم |
| `&` | reference (نه کپی) |
| `*` | pointer (آدرس) |
| `const` | تغییرناپذیر |
| `auto` | نوع رو خودت بفهم |
| `override` | دارم تابع کلاس پدر رو بازنویسی می‌کنم |
| `virtual` | این تابع می‌تونه override بشه |
| `<T>` | template — این کلاس/تابع با نوع T کار می‌کنه |

---

## ۱۵. چیزایی که لازم نیست یاد بگیری (هنوز)

- Memory management دستی (`new` / `delete`)
- Move semantics عمیق
- Template metaprogramming
- Concurrency و thread دستی (Drogon خودش مدیریت می‌کنه)
- Operator overloading

Drogon خیلی از پیچیدگی‌های C++ رو پشت API خودش مخفی کرده. همین که این فایل رو بدونی کافیه برای شروع.
