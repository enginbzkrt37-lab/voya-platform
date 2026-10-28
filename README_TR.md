# Voya Mobility Platform

Voya'nın tek HTML dosyasından daha ileri, sunucu tabanlı uygulama iskeleti. Müşteri, sürücü ve yönetici akışlarını tek projede toplar.

## Önemli gerçekçilik notu

Bu proje yalnızca statik demo değildir: Node.js API, PostgreSQL veritabanı, parola hashleme, JWT oturumu, rol kontrolleri, OTP doğrulama akışı, yolculuk kayıtları, sürücü konum endpoint'i, Socket.IO olayları, favoriler, yönetici metrikleri ve Stripe Checkout entegrasyon noktaları içerir.

Ancak gerçek dünyada hizmet verebilmesi için kendi servis hesapların ve yapılandırman gerekir:
- PostgreSQL veritabanı
- HTTPS kullanan Node.js sunucu barındırması
- Canlı SMS için Twilio Verify hesabı ve doğrulama servisi
- Kart ödemeleri için Stripe hesabı, sunucu anahtarı ve webhook
- Gerçek harita/rota/mesafe ve tarife hesaplaması için harita sağlayıcısı ve fiyatlandırma kuralları
- Gerçek sürücü ağı, sürücü kayıt/onay süreci, müşteri hizmetleri, yasal izinler ve güvenlik operasyonları

Bu anahtarlar/hesaplar olmadan SMS veya ödeme canlı olarak çalışmaz. Proje bunları varmış gibi göstermez. Bu sürüm canlı ücret hesaplamadığı için ödeme başlatmayı da ücret tanımlanmadan engeller.

## Gerekli ortam

- Node.js 20+
- PostgreSQL 14+
- npm

## Yerelde başlatma

1. Bu ZIP'i bilgisayarına çıkar.
2. Proje klasöründe terminal aç.
3. `npm install` çalıştır.
4. `.env.example` dosyasını `.env` adıyla kopyala.
5. PostgreSQL'de `voya` adlı veritabanı oluştur.
6. `.env` içindeki `DATABASE_URL` ve `JWT_SECRET` değerlerini ayarla. `JWT_SECRET` en az 32 karakterlik rastgele bir değer olmalı.
7. İlk yönetici hesabı için `ADMIN_EMAIL` ve güçlü `ADMIN_PASSWORD` belirle.
8. `npm start` çalıştır.
9. Tarayıcıda `http://localhost:3000` aç.

İlk açılışta tablolar otomatik oluşturulur. Yönetici hesabı yalnızca belirtilen e-posta veritabanında yoksa oluşturulur. Üretime almadan önce yönetici şifresini değiştir.

## SMS doğrulama

Twilio Verify için `.env` içine `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` ve `TWILIO_VERIFY_SERVICE_SID` gir. Sağlayıcı yapılandırılmadığında geliştirme ortamında OTP yalnızca sunucu konsoluna yazılır. `NODE_ENV=production` iken SMS ayarı yoksa kod gönderme isteği başarısız olur.

## Stripe ödeme

`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `APP_URL` ve `STRIPE_CURRENCY` ayarlanmalı. Stripe Dashboard'da webhook endpoint'ini `https://SENIN-DOMAININ/api/payments/webhook` olarak tanımla ve `checkout.session.completed` olayını aç. İmza doğrulaması yapılır. Güvenlik nedeniyle tarayıcıya gizli anahtar koyma.

Ödeme endpoint'i, sunucuda doğrulanmış `estimated_fare` olmayan yolculuklar için ödeme açmaz. Canlı fiyatlandırma ve ücret onayı eklenmeden ödeme özelliğini üretimde açma.

## Sürücü akışı

Yönetici/sunucu tarafından sürücü rolü atanmış hesapla giriş yapılır. Sürücü paneli talep almayı ve durum ilerletmeyi destekler. Konum paylaşımı tarayıcı konum izni ve HTTPS gerektirir. Sürücü hesabı oluşturmak için güvenli bir yönetici akışı ve sürücü kimlik/evrak kontrolü eklenmelidir; herkese sürücü rolü açan bir kayıt akışı özellikle yoktur.

## Üretim öncesi kontrol listesi

- [ ] Gerçek domain ve HTTPS
- [ ] PostgreSQL yedekleme, bağlantı havuzu ve izleme
- [ ] Güçlü, gizli ortam değişkenleri; `.env` dosyasını Git'e yükleme
- [ ] SMS sağlayıcısı ve teslimat testi
- [ ] Harita/rota/mesafe API'si, adres doğrulama ve gerçek tarife motoru
- [ ] Sürücü onboarding, belge/kimlik ve araç kontrolü
- [ ] Ödeme sağlayıcısı, webhook, iade/iptal ve mutabakat testleri
- [ ] Gizlilik/KVKK metinleri, açık rıza, veri saklama/silme süreçleri
- [ ] Rate-limit, güvenlik testleri, log/uyarı, olay müdahale planı
- [ ] Uçtan uca testler ve gerçek cihazlarda test

## API özeti

- `POST /api/auth/register`, `POST /api/auth/login`
- `POST /api/auth/send-otp`, `POST /api/auth/verify-otp`
- `GET /api/me`
- `POST /api/rides`, `GET /api/rides`, `PATCH /api/rides/:id/status`
- `POST /api/driver/location`, `GET /api/drivers/nearby`
- `GET/POST /api/favorites`, `DELETE /api/favorites/:id`
- `POST /api/payments/checkout`, `POST /api/payments/webhook`
- `GET /api/admin/overview`, `GET/PATCH /api/admin/users`, `GET /api/admin/audit`
- `GET /api/health`

## Dosya yapısı

- `public/index.html`: arayüz
- `public/styles.css`: tasarım ve mobil uyumluluk
- `public/app.js`: istemci etkileşimleri
- `server/index.js`: API, oturum, roller, SMS, ödeme, socket olayları
- `server/db/schema.sql`: PostgreSQL şeması
- `.env.example`: gerekli ayarlar
