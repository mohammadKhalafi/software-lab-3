# software-lab-3

# بخش اول - کشف خطا

ابتدا با اجرای تست های همه ی تست ها پاس می شوند:

![img.png](img.png)

## پرسش اول

پس از بررسی کد متوجه شدیم که در اجرای تابع محاسبه موجودی، توجهی به منفی شدن موجودی نمی شود و ممکن است موجودی حساب پس از چند تراکنش کمتر از صفر شود و به این نکته توجهی نشده است.

## پرسش دوم

برای این مشکل، تست زیر را می نویسیم (فرض می کنیم در برنامه درست اگر مقدار موجودی در هر زمان منفی شد برنامه `exception` می دهد.

![img_5.png](img_5.png)

در تصویر زیر مشاهده می کنیم که این تست پاس نشده است:

![img_2.png](img_2.png)

حال کد را به صورت زیر تغییر می دهیم تا این مشکل حل شود:

![img_4.png](img_4.png)

و مشاهده می کنیم که همه تست ها پاس می شوند:

![img_1.png](img_1.png)

## پرسش سوم

نوشتن تست بعد از نوشتن برنامه ممکن ایجاد بایاس ذهنی کند و تنها تست هایی نوشته شود که برنامه در آن ها درست کار می کند. همچنین پیش فرض های مسئله ممکن است در پیاده سازی فراموش شوند چون در زمان کد زدن به حالت های خاص توجه نمی کنیم.


# بخش دوم - به کارگیری TDD

## پاس کردن تست های اول و دوم 

با صدا زدن متد
addTransaction
در متد
calculateBalance
مشکلات تست های کامنت شده اول و دوم رفع شدند، چرا که قبل از ویزایش تراکنش ها در history ثبت نمیشدند.

## پاس کردن تست سوم

تست سوم را از حالت کامنت خارج و تست ها را اجرا کردیم. مشاهده می شود که تست آخر fail می شود: 


![Screenshot 2025-04-22 155956](https://github.com/user-attachments/assets/efc6c1e5-3879-4961-9fa8-fd0a089ac158)


### علت شکست تست:
این تست انتظار دارد که تاریخچه‌ ی تراکنش‌ ها فقط شامل آخرین مجموعه تراکنش‌ هایی باشد که به متد `(...)calculateBalance` داده شده‌اند. اما در پیاده‌ سازی اولیه، لیست `transactionHistory` به صورت  accumulative پر می‌ شد و تراکنش‌ های قبلی در آن باقی می‌ ماندند. در واقع تست `testTransactionHistoryShouldContainOnlyLastCalculationTransactions` انتظار دارد که وقتی متد `()calculateBalance` فراخوانی می‌ شود، نه‌ تنها موجودی صحیح بازگردانده شود، بلکه **تراکنش‌های ورودی نیز در تاریخچه‌ی تراکنش‌ها ثبت شوند**.

با بررسی فایل `AccountBalanceCalculator.java` مشخص شد که متد `()calculateBalance` فقط عملیات محاسبه‌ی موجودی را انجام می‌دهد و **تاریخچه‌ی تراکنش‌ها را به‌روزرسانی نمی‌کند**.

---

##  مرحله دوم: اصلاح کد

برای رفع این مشکل، در ابتدای متد `(...)calculateBalance`، لیست `transactionHistory` را پاک کردیم تا فقط تراکنش‌ های مربوط به محاسبه‌ ی فعلی در آن باقی بمانند. با اعمال این تغییر، هر بار که `()calculateBalance`  فراخوانی می‌شود، تاریخچه‌ی قبلی پاک شده و فقط تراکنش‌های مربوط به محاسبه‌ی فعلی در تاریخچه ثبت می‌شوند.

این دقیقاً رفتاری است که تست انتظار آن را دارد و باعث پاس شدن تست می‌شود.


###  تغییرات اعمال‌ شده در `AccountBalanceCalculator.java`:


```java
public static int calculateBalance(List<Transaction> transactions) {
    
     clearTransactionHistory(); 

    int balance = 0;
    for (Transaction t : transactions) {

        if (t.getType() == TransactionType.DEPOSIT) {
            balance += t.getAmount();
        } else if (t.getType() == TransactionType.WITHDRAWAL) {
            if (balance - t.getAmount() < 0)
                throw new IllegalStateException("Balance cannot be negative!");
            balance -= t.getAmount();
        }
        addTransaction(t);
    }
    return balance;
}
```


### اجرا مجدد تست ها :


مجددا تست ها را ران می کنیم و مشاهده می شود که تست آخر این بار با اعمال تغییرات گفته شده، پاس شد.

![Screenshot 2025-04-22 162209](https://github.com/user-attachments/assets/42bdeefe-8336-45cd-88fc-2f8048a69f30)

