Oracle Wallet یک قابلیت امنیتی در Oracle است که به ذخیره و مدیریت امن اطلاعات حساس مانند گواهی‌نامه‌ها (credentials)، کلیدهای رمزنگاری و دیگر داده‌های امنیتی کمک می‌کند. Oracle Wallet بخشی از مجموعه ابزارهای امنیتی Oracle Advanced Security است و به کاربران اجازه می‌دهد که اطلاعات حساس را در یک مخزن امن و رمزنگاری شده نگه دارند.

### قابلیت‌ها و مزایای Oracle Wallet

1. **امنیت بالا**: اطلاعات حساس مانند نام کاربری و رمز عبور به صورت رمزنگاری شده ذخیره می‌شوند، بنابراین امنیت داده‌ها تضمین می‌شود.
2. **مدیریت آسان**: به راحتی می‌توان اطلاعات کاربری، گواهی‌ها و کلیدهای رمزنگاری را مدیریت کرد.
3. **یکپارچگی با Oracle Database**: Oracle Wallet به خوبی با پایگاه داده Oracle و دیگر ابزارهای Oracle یکپارچه می‌شود.
4. **پشتیبانی از SSL/TLS**: برای مدیریت گواهی‌های SSL/TLS مورد استفاده قرار می‌گیرد که برای امنیت ارتباطات شبکه‌ای ضروری است.
5. **احراز هویت بدون رمز عبور**: امکان استفاده از احراز هویت قوی بدون نیاز به وارد کردن رمز عبور در هر بار اتصال به پایگاه داده.

### مراحل راه‌اندازی Oracle Wallet

#### 1. ایجاد Wallet

ابتدا باید Oracle Wallet را ایجاد کنید. برای این کار از ابزار `orapki` استفاده می‌شود.

```shell
orapki wallet create -wallet /path/to/wallet -pwd wallet_password
```

#### 2. اضافه کردن اطلاعات کاربری به Wallet

با استفاده از `orapki` می‌توانید اطلاعات کاربری را به Wallet اضافه کنید.

```shell
orapki wallet add -wallet /path/to/wallet -dn "CN=oracle_user" -pwd wallet_password -type user -user oracle_user -password oracle_password
```

#### 3. پیکربندی Oracle Net Services برای استفاده از Wallet

برای استفاده از Oracle Wallet در اتصال‌های پایگاه داده، فایل `sqlnet.ora` را به صورت زیر پیکربندی کنید:

```plaintext
SQLNET.WALLET_OVERRIDE = TRUE
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /path/to/wallet)))
```

#### 4. استفاده از Wallet در اتصال‌ها

در فایل‌های پیکربندی مانند `tnsnames.ora` و `listener.ora`، اطلاعات لازم را وارد کنید تا از Oracle Wallet استفاده شود.

```plaintext
MYDB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = myhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVICE_NAME = myservice)
    )
    (SECURITY =
      (SSL_SERVER_CERT_DN = "CN=oracle_user")
    )
  )
```

### یکپارچه‌سازی Oracle Wallet با Oracle GoldenGate

Oracle GoldenGate می‌تواند به جای ذخیره مستقیم نام کاربری و رمز عبور در فایل‌های پیکربندی از Oracle Wallet استفاده کند.

#### 1. پیکربندی Oracle GoldenGate برای استفاده از Wallet

در فایل پیکربندی Extract، Replicat و غیره، از پارامتر `USERIDALIAS` استفاده کنید.

```plaintext
EXTRACT ext1
USERIDALIAS ggs_user
EXTTRAIL ./dirdat/et
TABLE schema_name.table_name;
```

#### 2. پیکربندی فایل `sqlnet.ora` برای Oracle GoldenGate

Oracle GoldenGate باید بتواند از Oracle Wallet استفاده کند، بنابراین فایل `sqlnet.ora` باید به درستی پیکربندی شود.

```plaintext
SQLNET.WALLET_OVERRIDE = TRUE
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /path/to/wallet)))
```

با این تنظیمات، Oracle GoldenGate می‌تواند به صورت امن و بدون نیاز به ذخیره نام کاربری و رمز عبور به پایگاه داده متصل شود، که این امر امنیت سیستم را به طور قابل توجهی افزایش می‌دهد.
