
# Sensor Server on Railway (Flask)

یک سرور ساده برای دریافت داده‌های سنسور از طریق ماژول سیم‌کارت (GPRS) و نمایش در داشبورد.

## اجرا به صورت محلی
```bash
python -m venv .venv
source .venv/bin/activate  # ویندوز: .venv\Scripts\activate
pip install -r requirements.txt
export API_KEY=changeme  # ویندوز: set API_KEY=changeme
python app.py
```
سپس مرورگر: http://127.0.0.1:5000

تست درج داده:
```bash
curl "http://127.0.0.1:5000/data?key=changeme&temp=24.5&hum=60&soil=40&pump=0&fan=1"
```

## دیپلوی روی Railway
1. این پروژه را در گیت‌هاب آپلود کنید.
2. در Railway گزینه **New Project → Deploy from GitHub**.
3. در تب **Variables** مقدار `API_KEY` را ست کنید: یک رشته‌ی تصادفی و طولانی، مثلاً خروجی
   `python -c "import secrets; print(secrets.token_urlsafe(32))"`.
   بدون این متغیر، سرور هیچ داده‌ای قبول نمی‌کند (پاسخ 503). کلید را هیچ‌وقت داخل کد یا گیت‌هاب نگذارید.
4. پس از دیپلوی، URL عمومی شما آماده است.

## ارسال از طریق ماژول SIM800/SIM900 (GPRS HTTP GET)
(APN را با اپراتور خود جایگزین کنید.)
```
AT+CPIN?
AT+CREG?
AT+SAPBR=3,1,"CONTYPE","GPRS"
AT+SAPBR=3,1,"APN","YOUR_APN"
AT+SAPBR=1,1
AT+SAPBR=2,1

AT+HTTPINIT
AT+HTTPPARA="CID","1"
AT+HTTPPARA="URL","https://YOUR-APP.up.railway.app/data?key=YOUR_KEY&temp=24.5&hum=60&soil=40&pump=0&fan=1"
AT+HTTPACTION=0   // 0=GET, 1=POST
AT+HTTPREAD
AT+HTTPTERM

AT+SAPBR=0,1
```

## مسیرها (Endpoints)
- `GET/POST /data` : دریافت داده (`temp`، `hum`، `soil`، `pump`، `fan` و اختیاری `ts`). کلید در پارامتر `key` یا هدر `X-API-Key` لازم است.
- `GET /data/latest` : آخرین رکورد به صورت JSON.
- `GET /data/all` : همه‌ی رکوردها به صورت JSON.
- `GET /` : داشبورد گرافیکی.
- `GET /health` : وضعیت سرویس.

> نکته: برای پایداری بلندمدت بهتر است دیتابیس PostgreSQL استفاده شود. در نسخه ساده از SQLite استفاده شده است.
