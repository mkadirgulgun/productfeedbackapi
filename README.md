# ProductFeedback API

Kullanıcıların ürün/özellik geri bildirimlerini paylaşabildiği, oy verebildiği ve yorumlayabildiği **RESTful** bir API. Modern .NET yığını ile yazıldı; temiz katmanlar, DTO kullanımı ve dosya yükleme (ör. ekran görüntüsü) desteği bulunur.

- **Canlı Swagger:** feedback.mkadirgulgun.com.tr/index.html  
- **Test Kullanıcısı:** `test@mkadirgulgun.com` / `123Test..` *(yalnızca deneme amaçlıdır, periyodik olarak sıfırlanabilir)*

## İçindekiler
- [Özellikler](#özellikler)
- [Mimari & Teknolojiler](#mimari--teknolojiler)
- [Proje Yapısı](#proje-yapısı)
- [Hızlı Başlangıç](#hızlı-başlangıç)
- [Ortam Değişkenleri](#ortam-değişkenleri)
- [Göçler (Migrations)](#göçler-migrations)
- [Örnek İstek/Yanıtlar](#örnek-istekyanıtlar)
- [Yol Haritası](#yol-haritası)
- [Katkı](#katkı)
- [Lisans](#lisans)

## Özellikler
- 👤 **Kimlik Doğrulama** (JWT/Token tabanlı tasarıma uygun): kayıt, giriş, yenileme token akışına hazır.
- 📝 **Geri Bildirimler**: oluşturma, listeleme, detay, güncelleme, silme.
- 👍 **Oy/Beğeni**: bir geri bildirim için tekil kullanıcı oyu.
- 💬 **Yorumlar**: geri bildirim altında yorum akışı.
- 🏷️ **Kategoriler/Etiketler**: filtrelenebilir geri bildirim listeleri.
- 📎 **Dosya Yükleme**: `wwwroot/uploads` altında görselleri saklama.
- 🔍 **Swagger UI** ile keşfedilebilir uç noktalar.

## Mimari & Teknolojiler
- **ASP.NET Core Web API** (.NET 8+ uyumlu)
- **Entity Framework Core** ile veri erişimi
- **DTO** katmanı ile istek/yanıt sözleşmeleri
- **Model Validations** (action seviyesinde doğrulama için hazır yapı)
- **Swagger/OpenAPI** dokümantasyonu

## Proje Yapısı
productfeedbackapi/  
├─ Controllers/        # API uç noktaları  
├─ DTO/                # İstek/yanıt modelleri  
├─ Data/               # DbContext ve konfigürasyonlar  
├─ Model/              # Varlıklar (Entities)  
├─ Migrations/         # EF Core göçleri  
├─ wwwroot/uploads/    # Yüklenen dosyalar  
├─ Program.cs          # Uygulama başlangıcı  
├─ appsettings*.json   # Konfigürasyon  

## Hızlı Başlangıç
### Gereksinimler
- [.NET SDK 8.x](https://dotnet.microsoft.com/)
- Bir veritabanı (SQL Server veya PostgreSQL — projede EF Core ile konfigüre edilebilir)

### Çalıştırma
1) Depoyu klonla  
git clone https://github.com/mkadirgulgun/productfeedbackapi.git  
cd productfeedbackapi  

2) Gerekliyse appsettings.json'u doldur  

3) Veritabanını hazırla (EF Core migrations ile)  
dotnet restore  
dotnet build  
dotnet ef database update  

4) Uygulamayı çalıştır  
dotnet run  
Swagger UI -> http://localhost:5000/swagger (veya uygulamanın verdiği port)

## Ortam Değişkenleri
appsettings.json veya ortam değişkenlerinde aşağıdaki anahtarları tanımlayın (örnek):  
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=ProductFeedbackDb;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Jwt": {
    "Issuer": "ProductFeedback",
    "Audience": "ProductFeedback",
    "Key": "super-secret-key-change-me",
    "AccessTokenMinutes": 60,
    "RefreshTokenDays": 7
  },
  "Upload": {
    "RootPath": "wwwroot/uploads"
  }
}

## Göçler (Migrations)
Yeni bir özellik eklediğinizde:  
dotnet ef migrations add Add_New_Feature  
dotnet ef database update

## Örnek İstek/Yanıtlar
Swagger UI üzerinden tüm uç noktaları canlı test edebilirsiniz.

### 1) Kayıt Ol
POST /api/auth/register  
Content-Type: application/json  
{
  "email": "user@example.com",
  "password": "Password123!",
  "fullName": "Ada Lovelace"
}
200 OK  
{ "message": "Registered", "userId": "..." }

### 2) Giriş Yap
POST /api/auth/login  
Content-Type: application/json  
{ "email": "test@mkadirgulgun.com", "password": "123Test.." }
200 OK  
{
  "accessToken": "eyJhbGciOi...",
  "refreshToken": "....",
  "expiresIn": 3600
}

### 3) Geri Bildirim Oluştur
POST /api/feedbacks  
Authorization: Bearer <ACCESS_TOKEN>  
Content-Type: application/json  
{
  "title": "Karanlık tema",
  "description": "Uygulamada koyu mod istiyoruz.",
  "category": "UI/UX"
}
201 Created  
{
  "id": 42,
  "title": "Karanlık tema",
  "description": "Uygulamada koyu mod istiyoruz.",
  "category": "UI/UX",
  "votes": 0,
  "commentsCount": 0,
  "createdAt": "2025-08-15T15:00:00Z"
}

### 4) Oy Ver
POST /api/feedbacks/42/votes  
Authorization: Bearer <ACCESS_TOKEN>  
{ "feedbackId": 42, "votes": 12 }

### 5) Yorum Ekle
POST /api/feedbacks/42/comments  
Authorization: Bearer <ACCESS_TOKEN>  
Content-Type: application/json  
{ "content": "Bu özellik harika olur!" }
201 Created  
{ "id": 7, "feedbackId": 42, "content": "Bu özellik harika olur!", "author": "Ada", "createdAt": "2025-08-15T15:05:00Z" }

## Yol Haritası
- [ ] Şifre sıfırlama/eposta doğrulama akışı  
- [ ] Admin panel uçları (kategoriler, moderasyon)  
- [ ] Arama & gelişmiş filtreleme (durum/kategori/etiket)  
- [ ] Paginasyon & sıralama parametreleri  
- [ ] Dosya türü/doğrulama iyileştirmeleri  
- [ ] Entegrasyon testleri ve GitHub Actions ile CI  

## Katkı
Katkılar memnuniyetle karşılanır!  
- Fork → Feature branch → PR açın.  
- Kod stili ve isimlendirmeleri mevcut yapıya uyacak şekilde koruyun.  
- Public endpoint’lerde DTO doğrulamalarını ve hata dönüşlerini eklemeyi unutmayın.

## Lisans
Henüz lisans belirtilmediyse **“All Rights Reserved”** olarak kabul edilir. Tercih ettiğiniz lisansı LICENSE dosyasıyla ekleyebilirsiniz.
