برای تضمین امنیت Oracle GoldenGate و اجرای تکثیر داده‌ها از سرور اصلی به سرور GoldenGate به صورت یک‌طرفه، باید چندین مرحله و راهکار امنیتی را دنبال کنید. هدف این است که از ذخیره اطلاعات کاربری حساس در سرور اصلی اجتناب کنید و در عین حال داده‌ها را به صورت امن به سرور GoldenGate منتقل نمایید. در ادامه مراحل و راهکارهای امنیتی مورد نیاز توضیح داده شده است:

### 1. استفاده از Oracle Wallet

#### ایجاد Oracle Wallet در سرور GoldenGate

Oracle Wallet به شما اجازه می‌دهد که اطلاعات کاربری و رمز عبور را به صورت امن ذخیره کنید. ابتدا باید Oracle Wallet را در سرور GoldenGate ایجاد کنید.

```shell
orapki wallet create -wallet /path/to/wallet -pwd wallet_password
```

#### اضافه کردن اطلاعات کاربری به Oracle Wallet

اطلاعات کاربری پایگاه داده منبع را به Oracle Wallet اضافه کنید.

```shell
orapki wallet add -wallet /path/to/wallet -dn "CN=oracle_user" -pwd wallet_password -type user -user oracle_user -password oracle_password
```

#### پیکربندی فایل `sqlnet.ora`

فایل `sqlnet.ora` را برای استفاده از Oracle Wallet پیکربندی کنید.

```plaintext
SQLNET.WALLET_OVERRIDE = TRUE
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /path/to/wallet)))
```

### 2. استفاده از Credential Store در GoldenGate

Credential Store یک مکانیزم دیگر برای ذخیره امن اطلاعات کاربری است که توسط Oracle GoldenGate استفاده می‌شود.

#### ایجاد Credential Store

Credential Store را با استفاده از دستور GGSCI ایجاد کنید.

```plaintext
GGSCI> ADD CREDENTIALSTORE
```

#### اضافه کردن اطلاعات کاربری به Credential Store

اطلاعات کاربری را به Credential Store اضافه کنید و از یک نام مستعار (alias) برای شناسایی استفاده کنید.

```plaintext
GGSCI> ALTER CREDENTIALSTORE ADD USER oracle_user PASSWORD oracle_password ALIAS ggs_user DOMAIN OracleGoldenGate
```

### 3. پیکربندی Oracle GoldenGate برای استفاده از Credential Store

در فایل‌های پیکربندی Extract و Replicat، از نام مستعار (alias) به جای اطلاعات کاربری مستقیم استفاده کنید.

```plaintext
EXTRACT ext1
USERIDALIAS ggs_user DOMAIN OracleGoldenGate
EXTTRAIL ./dirdat/et
TABLE schema_name.table_name;
```

### 4. ایجاد اتصالات امن (SSL/TLS)

برای افزایش امنیت ارتباطات بین سرور اصلی و سرور GoldenGate، از SSL/TLS استفاده کنید.

#### ایجاد و تنظیم گواهی‌های SSL

گواهی‌های SSL را ایجاد کرده و در Oracle Wallet ذخیره کنید.

```shell
orapki wallet add -wallet /path/to/wallet -trusted_cert -cert "path/to/ca_cert.pem" -pwd wallet_password
orapki wallet add -wallet /path/to/wallet -user_cert -cert "path/to/server_cert.pem" -pwd wallet_password
```

#### پیکربندی SSL در فایل‌های `sqlnet.ora`

فایل‌های `sqlnet.ora` را برای استفاده از SSL/TLS پیکربندی کنید.

```plaintext
SSL_VERSION = 1.0
SSL_CLIENT_AUTHENTICATION = TRUE
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /path/to/wallet)))
```

### 5. تنظیم حداقل مجوزهای لازم

اطمینان حاصل کنید که حساب کاربری مورد استفاده برای اتصال به پایگاه داده منبع حداقل مجوزهای لازم را دارد. این کار را با محدود کردن دسترسی‌ها انجام دهید تا فقط به جداول و عملیات مورد نیاز دسترسی داشته باشد.

```sql
CREATE USER ggs_user IDENTIFIED BY password;
GRANT CONNECT, RESOURCE TO ggs_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON schema_name.table_name TO ggs_user;
```

### 6. مانیتورینگ و گزارش‌دهی

راه‌اندازی سیستم‌های مانیتورینگ و گزارش‌دهی برای نظارت بر فعالیت‌های GoldenGate و شناسایی هر گونه فعالیت غیرمجاز یا مشکوک.

- **فعال‌سازی Auditing**: فعال‌سازی Auditing در Oracle Database برای ثبت تمام دسترسی‌ها و تغییرات داده‌ها.
- **استفاده از ابزارهای مانیتورینگ**: استفاده از ابزارهای مانیتورینگ مانند Oracle Enterprise Manager (OEM) برای نظارت بر فعالیت‌های GoldenGate.

### 7. محافظت از فایل‌های پیکربندی

مطمئن شوید که فایل‌های پیکربندی Oracle GoldenGate (مثل `GLOBALS`, `manager.prm`, `extract.prm`, `replicat.prm`) به صورت ایمن ذخیره و دسترسی به آنها محدود شده است.

```shell
chmod 600 /path/to/goldengate/dirprm/*
```

### 8. مدیریت به‌روزرسانی‌ها

اطمینان حاصل کنید که نرم‌افزار Oracle GoldenGate و پایگاه داده Oracle به‌روز هستند تا از هرگونه آسیب‌پذیری امنیتی جلوگیری شود.

### خلاصه

با پیروی از این راهکارها، می‌توانید امنیت Oracle GoldenGate را تضمین کنید و انتقال داده‌ها را به صورت یک‌طرفه از سرور اصلی به سرور GoldenGate بدون ذخیره اطلاعات کاربری حساس در سرور اصلی انجام دهید. این اقدامات به شما کمک می‌کنند تا محیطی امن‌تر و مقاوم‌تر در برابر تهدیدات ایجاد کنید.
