# n8n Hava Durumu Bildirim Projesi

Bu proje, n8n kullanarak belirli bir şehrin hava durumunu OpenWeatherMap API'sinden çeken ve sıcaklık 20°C'nin üzerindeyse dinamik bir mesaj oluşturan otomatik bir iş akışıdır (workflow).

## Özellikler
- **Schedule Trigger:** Her gün otomatik çalışma.
- **API Entegrasyonu:** OpenWeatherMap üzerinden anlık veri çekme.
- **Koşullu Mantık:** 20°C üzeri sıcaklık kontrolü.

## Nasıl Kullanılır?
1. Bu depodaki `workflow.json` dosyasını indirin.
2. Kendi n8n editörünüze gelin, boş bir sayfada sağ üst menüden **Import from File** diyerek bu JSON'ı yükleyin.
3. [OpenWeatherMap](https://openweathermap.org/) üzerinden ücretsiz bir API key alın.
4. HTTP Request node'undaki `appid` parametresine kendi API key'inizi ekleyin.