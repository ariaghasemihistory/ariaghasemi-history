# PROJECT_MAP — ARIAGH.VISITOR (ویزیتور)

> نقشه راه برای ادامه پروژه در چت جدید. **اول این فایل را کامل بخوان؛ فایل‌های کد را فقط وقتی لازم شد و فقط خطوط لازم را بخوان** (شماره خط‌ها پایین آمده). نمودارها: `docs/ARCHITECTURE.md` · تاریخچه نسخه‌ها: `docs/CHANGELOG.md` · درس‌ها و باگ‌ها: `docs/LESSONS.md` · متن گفتگوها: `docs/CHAT_HISTORY.md`.

## ۱. هدف پروژه
- افزونه وردپرس/ووکامرس برای **فروش عمده و ویزیتوری** (فروشنده‌های میدانی). اول برای برند «بن لی» (benlifood.ir) با نام benli-wholesale 1.1.0 ساخته شد؛ از 2.0 عمومی و قابل‌فروش با نام **ARIAGH.VISITOR — ویزیتور** (بدون سیستم لایسنس).
- سه پنل: **مدیر** (wp-admin)، **ویزیتور** (صفحه فرانت با ورود وردپرس)، **مشتری عمده** (صفحه فرانت با کد اختصاصی/OTP).
- پشته: PHP 7.4+ (کد بدون سینتکس جدیدتر)، WordPress 6+، WooCommerce 7+ (سازگار با HPOS)، JS خالص (بدون فریم‌ورک)، REST API داخلی `bws/v1`، فارسی/RTL، تقویم شمسی.

## ۲. وضعیت فعلی
- **نسخه: 2.3.1** — `BWS_VERSION` در `ariagh-visitor/ariagh-visitor.php:25`، **DB: 1.5.0** (`:26`). فایل نهایی: `releases/ariagh-visitor-2.3.1.zip`.
- کار می‌کند (روی سایت تستی محلی تست شده):
  - مشتری/کد/OTP، اعتبار و بدهی، سفارش عمده، لغو سفارش
  - پیامک (۲۶ درایور، الگو)، اسنپ‌پی، المنتور، ویزیت/پیگیری
  - **مرجوعی کالا** با ربات تلگرام/بله، گوگل‌درایو و پاک‌سازی هاست
- **با سرویس واقعی تست نشده:** ربات‌های واقعی تلگرام و بله، گوگل‌درایو واقعی، پنل پیامک واقعی، اسنپ‌پی واقعی، کش سرور مشتری، Nginx.
- مشکل گزارش‌شده کاربر که هنوز تأیید نشده:
  - در 2.3.0 کاربر گفت مرجوعی جدید خودکار نمی‌آید و فقط با دکمه «مرجوعی‌های باز» دیده می‌شود.
  - در 2.3.1 مسیرهای جایگزین ارسال و تور ایمنی هر ۵ دقیقه اضافه شد.
  - **نتیجه روی سایت واقعی هنوز گزارش نشده.**

## ۳. نقشه فایل‌ها (`ariagh-visitor/`)
نقطه ورود: **`ariagh-visitor.php`** — ثابت‌ها (:25-26)، require همه کلاس‌ها (:31-57)، فعال‌سازی (:59)، HPOS (:63)، `plugins_loaded` → init همه + درگاه + ادمین (:71-110).

### هسته `includes/`
| فایل | کار | نمادهای کلیدی (خط) |
|---|---|---|
| `helpers.php` (500) | تنظیمات، رمزنگاری، تاریخ شمسی، اعداد | `bws_default_settings`:7 (همه کلیدهای تنظیمات + پیش‌فرض)، `bws_opt`:149، `bws_update_settings`:155، `bws_encrypt/decrypt`:171/189، `bws_secret_mask`:217، `bws_jdate`:293، `bws_now`:313، `bws_is_manager`:340، `bws_lock`:366، `bws_rate_limited`:400 |
| `class-install.php` (347) | جداول، نقش‌ها، مهاجرت | `maybe_upgrade`:17 (مهاجرت‌ها: perms، retail_label، bot_recipients)، `create_tables`:79 (۱۰ جدول)، `protect_dirs`:65 |
| `class-perms.php` (114) | دسترسی ویزیتور | `schema`:12 (همه مجوزها)، `for_user`:65، `scope_allows`:97 (own/all/none) |
| `class-auth.php` (353) | توکن مشتری HMAC، OTP، ویزیتور | `make_token`:19، `current_customer`:43 (هدر `X-BWS-Token`)، `otp_*`:105-153، `can_view_customer`:219، `save_visitor`:258 |
| `class-rest.php` (1000) | همه مسیرهای REST اصلی | `routes`:24 (فهرست)، `public_settings`:136، customer_*:164-319، `staff_me`:355 (شامل `returns_pending`)، `staff_customer`:488 (پرچم‌های `can`) |
| `class-customers.php` (390) | CRUD مشتری | `generate_code`:40، `create`:106، `query`:267، `to_array`:330 |
| `class-orders.php` (795) | سفارش عمده و لغو | `set_origin`:28، `source_label`:39 (برچسب «* سفارش از پنل … *»)، `can_cancel`:145، `quote`:208، `create`:309، `status_changed`:572 (همگام‌سازی دفتر؛ سفارش جایگزین مرجوعی بدون اثر مالی) |
| `class-ledger.php` (222) | دفتر اعتبار (+ بدهی، − پرداخت/برگشت) | `add`:54، `recalc`:80، `record_payment`:90 |
| `class-products.php` (212) | قیمت عمده | `sync`:14، `wholesale_price`:133، `catalogue`:163 |
| `class-sms.php` (432) + `sms/class-sms-drivers.php` (1359) | پیامک و ۲۶ درایور | `send_event`:242، `bulk_*`:284-375؛ درایورها کلاس‌های `BWS_SMS_Driver_*` |
| `class-events.php` (342) | رویدادهای پیامک و متغیرها | `variables`:12، `all`:113 (شامل return_new/admin/visitor/decision) |
| `class-security.php` (96) | هدرها، no-cache برای REST | `rest_nocache`:52 (**رفع باگ کش 2.1**) |
| `class-frontend.php` (301) | شورت‌کدها و صفحات پنل | شورت‌کدها:9-12 (`bws_customer_panel`، `bws_visitor_panel`، `bws_code_login`، `bws_leaderboard`)، `customer_panel`:131، `visitor_panel`:204 |
| `class-cron.php`، `class-stats.php`، `class-visits.php`، `class-print.php`، `class-log.php`، `class-icons.php`، `class-elementor.php` | کران روزانه، آمار، ویزیت، چاپ 80mm/A5، گزارش فعالیت (`$labels`)، آیکن SVG، ویجت المنتور | — |
| `gateways/class-snapppay-*.php` | درگاه اقساطی اسنپ‌پی (self-CMS v2.1) | `eligibility`:102، `build_cart`:245، `confirm`:474، `reconcile`:604 |

### مرجوعی `includes/returns/` (2.2 تا 2.3.1)
| فایل | کار | نمادها (خط) |
|---|---|---|
| `class-returns.php` (1003) | منطق مرجوعی و REST آن | `init`:31 (کران ۵ دقیقه)، `eligible_orders`:99، `create`:165، `decide`:287، `replacement_order`:359، `cancel`:401، `process`:478 (پیامک → `BWS_Bot::deliver` → درایو)، `retry`:528 (تور ایمنی attempts=0)، `query`:618، `to_array`:657، `delivery`:699، `routes`:737، `rest_*`:778-989 |
| `class-bot.php` (756) | ربات: گیرندگان، قالب‌ها، ارسال، تعامل | `sanitize_recipient`:49، `recipients`:76، `in_scope`:103، `link_for`:112، `templates`:123، `vars`:151، `render`:230، `deliver`:287، **`send_view`:322** (عکس+زیرنویس و جایگزین‌ها)، `send_update`:401 (Reply)، وب‌هوک `routes/rest_webhook`:435/443، `poll`:460، `handle_update`:512، `open_list`:587، `on_callback`:624، `finish_reject`:698، `apply_mode`:722، `preview`:738 |
| `class-messenger.php` (380) | کلاینت Bot API تلگرام/بله | `cfg`:21، `call`:60 (JSON/multipart، 429)، `reply_to`:158، `send_photo`:188، `send_album`:203، `clear_keyboard`:225، `set_webhook`:232، `send_text`:246، `find_chats`:348 |
| `class-gdrive.php` (452) | گوگل‌درایو OAuth/Service Account | `token`:81 (JWT)، `root_folder`:142، **`upload`:199 (Resumable + MD5)**، `download`:256، `verify_files`:277، `test`:322 (رفت‌وبرگشت)، `connect/oauth_callback`:375/399 |
| `class-return-files.php` (398) | عکس‌ها: اعتبارسنجی، ذخیره، لینک امضاشده، پاک‌سازی | `validate`:55، `store`:96، `mode`:151، `send_list`:165، `serve`:222 (پروکسی درایو)، `to_array`:267، `cleanup_*`:283-334، `push_to_drive`:346 |
| `class-http-multipart.php` (50) | سازنده multipart | `form`:17، `related`:40 (دیگر برای درایو استفاده نمی‌شود) |

### ادمین `includes/admin/`
| فایل | کار | نمادها |
|---|---|---|
| `class-admin.php` (1191) | منو، پوسته، داشبورد، تنظیمات، ذخیره | `sections`:7 (منوی کناری)، `spa_tab`:68، `render`:114، `settings_tabs`:676، `page_settings`:690 (تب returns → `BWS_Admin_Returns::render`)، `save_settings`:935، `private_keys`:1054 (کلیدهای محرمانه که export نمی‌شوند) |
| `class-admin-returns.php` (412) | تب «مرجوعی و ربات‌ها» | `render`:45، `recipient_row`:180، `recipients_card`:203، `mode_card`:220، `templates_card`:241، `cleanup_card`:257، `guides`:271، `save`:317 |
| `class-admin-sms.php` (322)، `class-admin-help.php` (220) | پنل پیامک، راهنمای کامل | — |

### فرانت `assets/`
| فایل | کار | نمادها |
|---|---|---|
| `js/bws-core.js` (635) | API و UI مشترک (`window.BWSU`) | `U.api`:54 (ضدکش `_`)، `U.modal`:89، `U.orderCards`:158، `U.upload`:196، `U.compressImage`:221، `U.returnCards`:248، `U.gallery`:267، `U.returnDetail`:305، `U.returnForm`:327، `U.Shop`:469، `U.frame`:609 |
| `js/customer-panel.js` (252) | پنل مشتری | `shell`:100، `tab`:118، `returnsTab`:178، `newReturn`:203 |
| `js/visitor-panel.js` (787) | پنل ویزیتور و مدیر (در wp-admin هم با `CFG.inAdmin`) | `navItems`:37، `tab`:78، `renderCustomer`:359، `returnsTab`:579، `openReturn`:631، `decisionForm`:668، `bulkSms`:723 |
| `js/admin.js` (254) | ابزارهای صفحات تنظیمات | تست ربات/درایو، گیرندگان، پیش‌نمایش قالب، پاک‌سازی |
| `css/bws.css` (714) | همه استایل‌ها (متغیرها :3-29) | بخش‌ها: Returns :599، Returns settings :664، v2.3 در انتها |
| `*.min.*` | نسخه فشرده (فقط از روی فایل اصلی بساز) | — |

### بقیه
- `templates/panel.php`: قالب تمام‌صفحه پنل.
- `uninstall.php`: حذف داده فقط اگر `delete_data_on_uninstall` روشن باشد.
- `readme.txt`: تغییرات نسخه‌ها.
- آرشیو: `releases/*.zip` (benli 1.1.0، 2.0.0 تا 2.3.1).
- تست: `tests/harness-bot-send-view.php` (اجرا: `php tests/harness-bot-send-view.php`).

## ۴. معماری و جریان داده (جزئیات و نمودار: `docs/ARCHITECTURE.md`)
- **REST:** پنل‌ها به `/wp-json/bws/v1/*` درخواست می‌دهند.
  - مشتری با هدر `X-BWS-Token` (HMAC). ستاد با nonce وردپرس.
  - بررسی دسترسی: `BWS_REST::is_customer/is_staff/is_manager`، بعد `BWS_Perms::scope_allows`.
- **تنظیمات:** همه در یک option به نام `bws_settings` (autoload). خواندن با `bws_opt()`، نوشتن با `bws_update_settings()`.
  - رازها با `bws_encrypt`، و در فرم با ماسک `••••••••` (اگر ماسک برگشت، مقدار قبلی می‌ماند).
- **جداول** (`wp_bws_*`): customers، ledger، orders (kind: order/settlement/replacement)، visits، products، sms_log، activity، returns، return_items، return_files (+md5).
- **چرخه مرجوعی:**
  1. ثبت: REST create → `BWS_Returns::create` (قفل، اعتبارسنجی، ذخیره عکس روی هاست).
  2. `defer` → در shutdown بعد از پاسخ (`fastcgi_finish_request`) اجرای `process`: پیامک یک‌بار → `BWS_Bot::deliver` (برای هر گیرنده) → `push_to_drive`.
  3. شکست موقت → option `bws_returns_retry` → کران `bws_returns_retry` هر ۵ دقیقه.
- **وضعیت ارسال** در ستون `notify_state` (JSON):
  - `bots[event|platform|chat]` = 'ok' یا متن خطا
  - `root[platform|chat]` = message_id پیام اول (برای Reply)
  - `kb[...]` = پیام‌های دارای دکمه
  - `photo_error`، `sms_new`
- **ربات تعاملی:**
  - وب‌هوک `POST /bws/v1/bot/{telegram|bale}/{secret32}` (و هدر secret تلگرام)، یا polling با کران `bws_bot_poll` هر دقیقه.
  - `handle_update` → `on_message` / `on_callback`.
  - callback_data: `v:ID` (مشاهده)، `a:ID` (شروع تأیید)، `ar:ID:res` (تأیید نهایی)، `r:ID` (رد)، `x:ID` (انصراف).
  - حالت «منتظر دلیل رد» در transient `bws_bot_st_*`.
- **کران‌ها:** `bws_daily` (لغو سفارش‌های پرداخت‌نشده، یادآوری، هرس، پاک‌سازی خودکار عکس)، `bws_returns_retry` (۵ دقیقه)، `bws_bot_poll` (۱ دقیقه، فقط در حالت polling)، `bws_snapppay_reconcile`.

## ۵. تصمیم‌های کلیدی
- **REST + JS خالص** به جای admin-ajax/React: سبک، بدون build step، سازگار با هر قالب. (React رد شد.)
- **تنظیمات در یک option با autoload:** بدون کوئری اضافه در صفحات.
- **مشتری بدون حساب وردپرس، با کد اختصاصی و توکن HMAC:** ساخت کاربر وردپرس برای هر مشتری رد شد.
- **اعتبار به شکل دفتر حساب** (جمع ردیف‌ها، `balance` کش): برگشت مرجوعی = ردیف منفی نوع `return`.
- **مرجوعی:**
  - فقط از روی سفارش ثبت می‌شود؛ تعداد ≤ خریداری‌شده منهای مرجوعی‌های در انتظار یا تأییدشده.
  - نوع رسیدگی (اعتبار/جایگزین/دستی) را تأییدکننده انتخاب می‌کند.
  - سفارش جایگزین = سفارش ووکامرس با مبلغ صفر، meta `_bws_replacement` و kind=replacement، بدون اثر مالی.
- **اعلان‌ها بعد از پاسخ HTTP اجرا می‌شوند** تا کاربر منتظر تلگرام/درایو نماند. تلاش مجدد فقط برای خطاهای موقت است (نه 400/401/403).
- **ربات:** گیرنده‌ها جداگانه‌اند (`bot_recipients`)، نه لیست ساده شناسه‌ها.
  - ویزیتور با محدوده own فقط مشتریان خودش را می‌بیند. لینک ویزیتور به visitor-panel و لینک مدیر به wp-admin می‌رود.
  - تأیید/رد داخل ربات با هویت کاربر وردپرسِ متصل و همان دسترسی‌های پنل انجام می‌شود. در گروه، شناسه شخصی فشارنده چک می‌شود.
  - تلگرام HTML می‌گیرد؛ بله متن ساده. `**x**` در قالب برای تلگرام `<b>` می‌شود و برای بله حذف.
- **گوگل‌درایو:**
  - هر دو روش اتصال: OAuth با scope `drive.file` (افزونه پوشه خودش را می‌سازد) و Service Account که **Shared Drive لازم دارد**.
  - آپلود Resumable + تطبیق MD5. multipart/related کنار گذاشته شد.
- **محل ذخیره عکس:** host / gdrive / both.
  - عکس‌های هاست در `uploads/ariagh-visitor-private/Y/m/<random32>.jpg` با `.htaccess` deny و لینک امضاشده ۲ ساعته.
  - عکس‌ها در مرورگر فشرده می‌شوند (۱۶۰۰px) و روی سرور دوباره ساخته می‌شوند (۲۰۰۰px، q82، EXIF حذف).
- **هاست کاربر خارج از ایران است:** تلگرام و گوگل مستقیم وصل می‌شوند. فیلد API Base/Relay برای سایر سایت‌ها نگه داشته شد.

## ۶. راه‌های امتحان‌شده و شکست‌خورده
- **آپلود multipart/related به درایو:** کاربر گزارش داد عکس‌ها در درایو نه باز می‌شوند و نه بعد از دانلود. بدنه را با parser پایتون تست کردم و سالم بود، پس علت قطعی پیدا نشد.
  - جایگزین: Resumable + MD5 + دکمه «آزمایش کامل» (آپلود، دانلود، مقایسه) + «بررسی سلامت».
  - عکس‌های خراب قدیمی در حالت «فقط درایو» قابل تعمیر نیستند، چون نسخه هاستشان حذف شده.
- **پیام جدا برای متن و آلبوم عکس (2.2):** کاربر نخواست. حالا عکس + زیرنویس در یک پیام است.
  - آلبوم (sendMediaGroup) دکمه نمی‌پذیرد؛ دکمه‌ها در یک Reply کوچک زیر آلبوم می‌آیند.
  - زیرنویس حداکثر ۱۰۲۴ حرف است؛ `render()` اول فیلدهای بلند را کوتاه می‌کند.
- **فقط یک مسیر برای ارسال عکس (2.3.0):** اگر بله/تلگرام آن را با 400 رد می‌کرد، هیچ چیز نمی‌رسید. 2.3.1 همیشه جایگزین متنی دارد. **هیچ مسیر ارسالی نباید بدون جایگزین باشد.**
- **thumbnail عمومی درایو** (`drive.google.com/thumbnail?id=`) برای فایل‌های خصوصی کار نمی‌کند. حالا تصویر کوچک از طریق سایت دانلود و نمایش داده می‌شود (`serve()`).
- **`getUpdates` برای «یافتن شناسه‌ها»** وقتی وب‌هوک فعال است کار نمی‌کند. حالا چت‌هایی که پیام داده‌اند در `bws_bot_seen_{p}` ذخیره می‌شوند و /start شناسه را به خود شخص می‌گوید.
- **درخواست‌های GET بدون پارامتر ضدکش (2.0):** LiteSpeed/CDN پاسخ کهنه می‌دادند (لیست خالی، مشتری حذف‌شده هنوز دیده می‌شد).
- **دریافت مستقیم فایل با curl از پشت پراکسی محیط کار Claude:** دامنه‌های wordpress.org، core.telegram.org و docs.bale.ai بسته بودند. برای مستندات از WebFetch استفاده شد. محیط تست وردپرس هم با ریست کانتینر از بین رفت.

## ۷. قوانین و ترجیحات کاربر
- زبان: فارسی، RTL، اعداد فارسی در UI، تاریخ شمسی. واحد پول از ووکامرس.
- برند: **ARIAGH.VISITOR — «ویزیتور»** با لوگوی هم‌سبک ARIAGH.AIWRITER. باید برای هر سایتی کار کند (نه فقط بن لی) و تمیز و قابل فروش باشد. سیستم لایسنس ندارد.
- کاربر **«بدون باگ»** می‌خواهد و انتظار دارد مستندات رسمی (بله، تلگرام، گوگل) خوانده شوند. قابلیت‌های کاربردی اضافه هم دوست دارد.
- برای درخواست‌های بزرگ اول سؤال بپرس (AskUserQuestion) و بعد پیاده کن. برای اصلاحات کوچک مستقیم انجام بده.
- **نسخه‌بندی:**
  - X.Y.0 برای قابلیت، X.Y.Z برای اصلاح.
  - با هر نسخه این‌ها را به‌روز کن: `Version:` در هدر، `BWS_VERSION`، `Stable tag` و Changelog در `readme.txt`.
  - با تغییر جدول، `BWS_DB_VERSION` را بالا ببر و مهاجرت را در `maybe_upgrade` بنویس.
- **بسته‌بندی:**
  1. `php -l` روی همه فایل‌ها.
  2. فشرده‌سازی با esbuild: `esbuild assets/js/X.js --minify --charset=utf8 --target=es2017 --outfile=assets/js/X.min.js` (و همین برای css). **`--charset=utf8` لازم است** تا متن فارسی سالم بماند.
  3. zip پوشه `ariagh-visitor` با نام `ariagh-visitor-X.Y.Z.zip`.
- **کد:**
  - پیشوند `bws_` / `BWS_`.
  - سازگار با PHP 7.4 (بدون `match`، `?->` یا named args).
  - escaping کامل، nonce/permission روی هر route، رازها رمزنگاری‌شده.
  - فراخوانی تلگرام، درایو و پیامک بعد از پاسخ HTTP.
- پاسخ نهایی به کاربر: کوتاه و فارسی. صادقانه بگو چه چیزی با سرویس واقعی تست نشده.

## ۸. باگ‌های حل‌شده که ممکن است تکرار شوند (کامل: `docs/LESSONS.md`)
1. **کش REST** (2.1): در `U.api` هر GET پارامتر `_` می‌گیرد و `cache:'no-store'` دارد؛ سرور هدر no-store، LiteSpeed و CDN می‌فرستد. بعد از نصب، کش سایت باید پاک شود.
2. **`[hidden]` با CSS قالب بازنویسی می‌شد:** `.bws-ui [hidden], .bws-modal-wrap [hidden]{display:none!important}`. مودال‌ها به body اضافه می‌شوند، پس داخل `.bws-ui` نیستند.
3. **ستون Origin ووکامرس «Unknown» نشان می‌داد:** با `BWS_Orders::set_origin` (متای `_wc_order_attribution_*`) رفع شد.
4. **سفارش جایگزین مرجوعی نباید دفتر اعتبار را تغییر دهد:** در `status_changed` شاخه زودهنگام برای `_bws_replacement` وجود دارد.
5. **rate limit ثبت مرجوعی 12 در ساعت کم بود** (خطاهای اعتبارسنجی هم شمرده می‌شدند)؛ به 30 رسید.
6. **onclick مشترک:** لیست مرجوعی داخل زیرتب مشتری `onclick` سفارش‌ها را بازنویسی می‌کرد. حالا کانتینر جدا دارد و هندلر سفارش فقط `[data-order]` را می‌گیرد.
7. **select یا checkbox با `disabled` ارسال نمی‌شود:** در `sanitize_recipient` پیش‌فرض‌ها امن‌اند (scope=all، decide=0).
8. **ردیف‌های تنظیمات در ستون باریک له می‌شدند:** `.bws-returns-settings .bws-row{grid-template-columns:1fr}`. select غیرفعال الگوی تکراری داشت؛ `background-repeat:no-repeat`.
9. **ارسال عکس بدون جایگزین** (2.3.1): `send_view` جایگزین متنی دارد. خطاهای 401، 403، 429 و شبکه «سخت» هستند و فقط در صف تلاش مجدد می‌روند.
10. **ارسال هیچ‌وقت اجرا نشد** (پروسه کشته شد): `retry()` هر ۵ دقیقه درخواست‌های pending با attempts=0 را بعد از ۳ دقیقه دوباره پردازش می‌کند.

## ۹. مشکلات باز و قدم‌های بعدی (به ترتیب)
1. از کاربر نتیجه 2.3.1 روی سایت واقعی را بگیر: آیا اعلان خودکار هر مرجوعی برای همه گیرنده‌ها می‌رسد؟ اگر نه، متن «وضعیت ارسال به ربات‌ها» را در جزئیات همان مرجوعی (پنل مدیر) بخواه.
2. نتیجه «آزمایش کامل گوگل‌درایو» روی هاست واقعی را بگیر. اگر رفت‌وبرگشت خراب بود، مشکل از پراکسی یا فایروال هاست است.
3. در بله تأیید کن caption آلبوم، `reply_to_message_id`، `answerCallbackQuery` و وب‌هوک واقعاً کار می‌کنند (مستندات بله ناقص است).
4. برای کاربر کم‌بازدید: کران واقعی هاست برای `wp-cron.php` هر ۵ دقیقه.
5. اختیاری: نسخه thumbnail کوچک برای گالری (الان عکس ۱۶۰۰px کامل لود می‌شود).
6. محیط تست وردپرس + ووکامرس + SQLite را دوباره بساز و تست‌ها را بازنویسی کن (روش در `docs/TESTING.md`).

## ۱۰. راهنمای کار با این نقشه
- این نقشه را اول بخوان. فایل‌های کد را فقط وقتی لازم شد و فقط خطوط لازم را بخوان (grep نماد → Read با offset و limit).
- هیچ‌چیزِ بخش «شکست‌خورده» را تکرار نکن.
- بعد از هر تغییر مهم (نسخه جدید، فایل جدید، تصمیم، باگ رفع‌شده) بخش مربوط در همین نقشه را در همان نوبت با یک Edit کوچک به‌روز کن. `docs/CHANGELOG.md` و `docs/LESSONS.md` را هم به‌روز کن.
- اگر نقشه با کد اختلاف داشت، کد معتبر است؛ نقشه را اصلاح کن.
- وقتی چت طولانی شد، project-handoff را دوباره اجرا کن و نقشه را تازه کن.
