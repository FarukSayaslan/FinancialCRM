# Anthropic API Integration Guide

## Proje Yapýsý

Bu proje Anthropic Claude API'sini kullanarak kredi skorlama ve müþteri risk analizi yapan bir finansal CRM uygulamasýdýr.

## API Bilgileri

- **API Saðlayýcý**: Anthropic
- **Model**: claude-3-5-sonnet-20241022
- **API Key**: AQ.Ab8RN6JUJMFkG28z0pvd54c83lT60_NWGOXJGp-jqS3vt_fthw
- **Base URL**: https://api.anthropic.com/v1
- **Endpoint**: /messages

## Dosya Yapýsý

```
FinancialCrm/
??? Helpers/
?   ??? ApiClient.cs          # Anthropic API istemcisi
?   ??? AppConfig.cs          # Konfigürasyon yönetimi
?   ??? SecretManager.cs      # Gizli bilgileri yönetme
?   ??? DataRepository.cs     # CSV/JSON veri iþleme
?   ??? ColorTheme.cs         # UI tema renkleri
?
??? Services/
?   ??? CreditScoringService.cs   # Kredi skorlama iþlemi (AI + Local)
?
??? Forms/
?   ??? MainForm.cs           # Ana form
?   ??? CustomerInputForm.cs  # Müþteri giriþ formu
?   ??? ResultForm.cs         # Sonuç gösterimi
?   ??? ExplainabilityForm.cs # Faktör analizi
?   ??? ComparisonForm.cs     # AI vs Gerçek sonuç
?   ??? HistoryForm.cs        # Geçmiþ tahminler
?
??? Models/
?   ??? Customer.cs           # Müþteri modeli
?   ??? CreditResult.cs       # Skorlama sonucu
?
??? Data/
?   ??? customers.csv         # Müþteri verileri (100 kayýt)
?   ??? labels.csv            # Risk etiketleri
?   ??? predictions_log.json  # Tahmin geçmiþi
?
??? Examples/
?   ??? ApiUsageExample.cs    # Kullaným örnekleri
?
??? config.json               # Uygulama konfigürasyonu
```

## Kullaným

### 1. Konfigürasyonu Yükleme

```csharp
using FinancialCrm.Helpers;

// Konfigürasyonu yükle
AppConfig.LoadConfig("config.json");

// API anahtarýný al
var apiKey = AppConfig.GetAnthropicApiKey();
```

### 2. Kredi Skorlama Servisi Kullanma

#### a) Yerel Algoritma (Hýzlý, Çevrimdýþý)

```csharp
using FinancialCrm.Services;

var service = new CreditScoringService();

var result = service.CalculateCreditScoreLocal(
    monthlyIncome: 5000,
    totalDebt: 10000,
    creditHistory: 60,
    defaultCount: 0,
    assetValue: 120000
);

Console.WriteLine($"Puan: {result.CreditScore}");
Console.WriteLine($"Karar: {result.Decision}");
```

#### b) AI Kullanarak (Daha Akýllý, Async)

```csharp
using FinancialCrm.Services;
using System.Threading.Tasks;

var service = new CreditScoringService();

var result = await service.CalculateCreditScoreAsync(
    firstName: "Ahmet",
    lastName: "Yýlmaz",
    age: 35,
    monthlyIncome: 5000,
    totalDebt: 10000,
    creditHistory: 60,
    defaultCount: 0,
    assetValue: 120000
);

Console.WriteLine($"Puan: {result.CreditScore}");
Console.WriteLine($"Karar: {result.Decision}");
Console.WriteLine($"AI Kullandý: {result.IsAIPowered}");
```

### 3. Etkili Faktörleri Analiz Etme

```csharp
var factors = service.GetInfluencingFactors(
    monthlyIncome: 5000,
    totalDebt: 10000,
    creditHistory: 60,
    defaultCount: 0,
    assetValue: 120000
);

foreach (var factor in factors)
{
    Console.WriteLine($"{factor.Key}: {factor.Value}");
}
```

## API Ýsteði Örneði

### Request
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": "Müþteri bilgileri... Kredi skorlama yap"
    }
  ]
}
```

### Response
```json
{
  "content": [
    {
      "type": "text",
      "text": "{\n  \"creditScore\": 750,\n  \"decision\": \"Onay\",\n  \"recommendedLimit\": 25000,\n  \"reasoning\": \"Düþük borç oraný ve uzun kredi geçmiþi.\"\n}"
    }
  ]
}
```

## Kredi Puanlama Kriterleri

### Yerel Algoritma

| Faktör | Puan Etki |
|--------|----------|
| Borç/Gelir Oraný < 0.3 | +150 |
| Borç/Gelir Oraný < 0.5 | +100 |
| Kredi Geçmiþi > 60 ay | +150 |
| Temerrüt Sayýsý = 0 | +100 |
| Varlýk Deðeri > 2 yýl gelir | +100 |

### Karar Verme

- **700+**: Onay ?
- **500-699**: Ýnceleme Bekliyor ?
- **<500**: Ret ?

## Önemli Notlar

1. **API Key Güvenliði**
   - API anahtarýný asla public repository'lerde paylaþmayýn
   - Production ortamýnda environment variables kullanýn
   - `SecretManager` sýnýfýný kullanarak anahtarlarý maskeleme

2. **Fallback Mekanizmasý**
   - AI API baðlantý sorununda yerel algoritma devreye girer
   - Böylece uygulama her zaman çalýþabilir durumda kalýr

3. **Rate Limiting**
   - Anthropic API kullaným sýnýrlarýný kontrol edin
   - Production ortamýnda request batching kullanýn

4. **Veri Güvenliði**
   - Müþteri verilerini þifrelenmiþ þekilde sakla
   - API yanýtlarýný logla ve analiz et

## Örnek Çalýþtýrma

```csharp
// Program.cs veya Form_Load olayýnda

AppConfig.LoadConfig();

// Asenkron metodu çaðýr
var result = await new CreditScoringService().CalculateCreditScoreAsync(
    "Ahmet", "Yýlmaz", 35, 5000, 10000, 60, 0, 120000
);

MessageBox.Show($"Kredi Puaný: {result.CreditScore}\nKarar: {result.Decision}");
```

## Sorun Giderme

### "API çaðrýsý baþarýsýz" hatasý
- API anahtarýný kontrol edin
- Ýnternet baðlantýsýný kontrol edin
- API endpoints'in doðru olup olmadýðýný kontrol edin

### JSON parse hatasý
- API yanýt formatýný kontrol edin
- Encoding ayarlarýný kontrol edin (UTF-8)

### Timeout hatasý
- Network gecikmesini artýrýn
- Request timeout deðerini ayarlayýn

## Ýletiþim ve Destek

Sorularýnýz veya sorunlarýnýz olmasý durumunda:
- Loglama verilerini kontrol edin
- ExceptionHandler'ý kullanýn
- API dokumentasyonuna baþvurun

---

**Sürüm**: 1.0.0
**Son Güncelleme**: 2024
**API Modeli**: claude-3-5-sonnet-20241022
