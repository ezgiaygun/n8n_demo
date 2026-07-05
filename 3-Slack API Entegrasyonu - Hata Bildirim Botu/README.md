# n8n API Call Hata Bildirim Botu

Bu n8n projesi, belirli bir API'ye (veya harici bir servise) atılan isteklerin başarısız olması (HTTP durum kodunun `200` olmaması) durumunda otomatik olarak devreye giren ve **Slack** kanalına anlık, dinamik hata bildirimleri gönderen bir otomasyon akışıdır.

## Özellikler

- **Dinamik Parametre Yönetimi:** Dışarıdan gelen `POST` isteklerindeki `body.userId` parametresini alarak dinamik API çağrıları yapar.
- **Hata Toleransı (Never Fail):** API çökmelerinde veya `4xx/5xx` hatalarında n8n workflow'unun tamamen durmasını engeller ve hatayı güvenli bir şekilde `IF Node` üzerinden işler.
- **Akıllı Koşul Kontrolü:** HTTP Status Code `200` (OK) dışındaki tüm senaryoları (`404 Not Found`, `500 Internal Error` vb.) otomatik olarak hata kabul eder.
- **Slack Entegrasyonu:** Hata veren workflow'un adını, ID'sini, dönen durum kodunu ve API'den gelen ham hata mesajını içeren zenginleştirilmiş (Markdown destekli) bildirimler gönderir.

---

## Sistem Mimarisi ve İş Akışı

Workflow temelde şu 4 düğümden (node) oluşmaktadır:

1. **Webhook Node (POST):** Tetikleyici (Trigger) düğümdür. Dışarıdan `{"userId": 999}` gibi bir JSON payload'u kabul eder.
2. **HTTP Request Node:** `JSONPlaceholder` veya `HTTPBin` gibi harici bir servis üzerinden gelen `userId` ile sorgulama yapar. *Settings -> On Error -> Continue Regular Workflow* olarak ayarlanmıştır.
3. **If Node:** Gelen yanıtın içindeki `statusCode` değerini inceler. Koşul: `statusCode != 200`.
4. **Slack Node:** Koşul `true` (başarısız) olduğunda devreye girer ve ilgili kanala detaylı hata raporu fırlatır.

---

## Nasıl Kullanılır?
1. Bu depodaki `workflow.json` dosyasını indirin.
2. Kendi n8n editörünüze gelin, boş bir sayfada sağ üst menüden **Import from File** diyerek bu JSON'ı yükleyin.
3. Slack entegrasyonunuzu ekleyin.

## Kurulum ve Entegrasyon Adımları

### 1. Slack Bot Ayarları
1. [Slack API Apps](https://api.slack.com/apps) sayfasına gidin ve yeni bir uygulama oluşturun. (Örneğin: n8n Hata Botu)
2. **OAuth & Permissions** sekmesinden **Bot Token Scopes** alanına gelin ve `chat:write` iznini (scope) ekleyin.
3. Uygulamayı workspace'inize yükleyin ve üretilen `xoxb-...` bot token'ını kopyalayın.
4. Mesajların gönderilmesini istediğiniz Slack kanalına girip `/invite @Bot_Adınız` komutu ile botu kanala davet edin.

### 2. n8n Kimlik Bilgileri (Credentials)
- n8n arayüzünde Slack düğümünü açın.
- Authentication tipini `Access Token` seçin ve kopyaladığınız `xoxb-...` token'ını girerek bağlantıyı kaydedin.

---

## Test Etme

Workflow'un düzgün çalıştığını doğrulamak için webhook test URL'ine terminal üzerinden bir `POST` isteği fırlatabilirsiniz:

### Başarılı Senaryo (Hata Tetiklenmez)
Mevcut bir kullanıcı (örn: `1`) sorgulandığında API `200 OK` döner ve Slack'e bildirim **gitmez**:
```bash
Invoke-RestMethod -Uri "http://<n8n-domain>/webhook-test/user" -Method Post -Headers @{"Content-Type"="application/json"} -Body '{"userId":"1"}'
```

### Hata Senaryosu (Slack Bildirimi Tetiklenir)
Mevcut olmayan hayali bir kullanıcı (örn: 999) sorgulandığında API `404 Not Found` döner ve Slack kanalınıza anında bildirim **düşer**:
```bash
Invoke-RestMethod -Uri "http://<n8n-domain>/webhook-test/user" -Method Post -Headers @{"Content-Type"="application/json"} -Body '{"userId":"999"}'
```