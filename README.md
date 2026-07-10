# Jeopolitik Haber Otomasyonu (n8n)

Popüler haber kaynaklarından (BBC, Guardian, Al Jazeera, AA vb.) jeopolitik gündemi çeken, **GDELT** ile doğrulayan, **Google Trends** ile trend analizi yapan ve sonucu **Telegram**'a raporlayan bir n8n workflow'u.

## Akış

1. **Kaynak toplama** — RSS feed'lerinden (BBC/Guardian/AlJazeera/AA world) ve Google Trends RSS'ten (`geo=TR`) paralel veri çekilir
2. **Kümeleme ve skorlama** — haberler ortak temalara göre kümelenir, kaynak sayısı + GDELT domain doğrulaması + trend verisiyle puanlanır
3. **Doğrulama** — her kümenin GDELT üzerinde kaç bağımsız kaynakta göründüğü kontrol edilir (rate limit: 1 istek/5sn)
4. **Fikir üretimi** — bir LLM (OpenRouter üzerinden) skorlanan kümelerden haber başlığı/fikri üretir
5. **Raporlama** — sonuçlar Telegram botu üzerinden gönderilir, buton ile onay/red verilebilir

## Gerekli kurulum

Workflow'u kendi n8n'inizde çalıştırmak için aşağıdaki ortam değişkenlerini (`.env` veya n8n credential olarak) tanımlamanız gerekir:

| Değişken | Açıklama | Zorunlu mu |
|---|---|---|
| `OPENROUTER_API_KEY` | Haber fikri/özeti üreten LLM çağrısı için | Evet |
| `TREND_BOT_TOKEN` | Telegram bot token (raporun gönderileceği bot) | Evet |
| `TREND_CHAT_ID` | Raporun gönderileceği Telegram chat ID | Evet |
| `TREND_WEBHOOK_SECRET` | `Calistir Webhook`'u tetikleyen isteğe eklenen paylaşılan sır (`X-Webhook-Secret` header'ı) | Evet |
| `N8N_PUBLIC_URL` | `Calistir Webhook`'un dışarıdan erişilebilir tam adresi (sadece localhost yeterliyse boş bırakılabilir) | Hayır |
| `REDDIT_CLIENT_ID` / `REDDIT_CLIENT_SECRET` | Sosyal skor için Reddit OAuth (opsiyonel) | Hayır |
| `GENERIC_TIMEZONE` | Zamanlanmış tetikleyici için (örn. `Europe/Istanbul`) | Önerilir |

> GDELT ve haber kaynağı RSS feed'leri (BBC/Guardian/AlJazeera/AA, Google Trends RSS) için API key gerekmez.

n8n'de env değişkenlerine node içinden erişebilmek için `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` ayarının açık olması gerekir.

## Kurulum adımları

1. n8n arayüzünde **Workflows → Import from File** ile `Jeopolitik-Haber-n8n-WORKFLOW.json` dosyasını içe aktarın
2. Yukarıdaki ortam değişkenlerini tanımlayın
3. **Data Table oluşturun:** n8n'de `trend_adaylar` adında bir Data Table oluşturun (kolonlar: tarih, baslik, aci, skor, kaynak_sayisi, trend_puani, sosyal_puani, durum, karar_zamani vb.) ve ID'sini `Adaylari Kaydet` ile `Karari Kaydet` node'larındaki `YOUR_DATA_TABLE_ID` placeholder'ının yerine yapıştırın
4. **Header Auth credential'ları oluşturun:** iki webhook node'unda (`Onay Webhook`, `Calistir Webhook`) `authentication: headerAuth` açıktır ama credential değerleri güvenlik nedeniyle dışa aktarılmamıştır — import sonrası n8n'de 2 ayrı "Header Auth" credential oluşturup ilgili node'lara atamanız gerekir:
   - `Onay Webhook` için header adı `X-Telegram-Bot-Api-Secret-Token`
   - `Calistir Webhook` için header adı `X-Webhook-Secret`, değeri `TREND_WEBHOOK_SECRET` env değişkeniyle aynı olmalı
5. Telegram bot webhook'unu ayarlayın: `allowed_updates` içinde `message` ve `callback_query` olmalı; `setWebhook` çağrısına `secret_token` parametresi olarak `Onay Webhook` credential'ındaki değerin aynısını ekleyin
6. Workflow'u **yayına alın (publish/active)** — production webhook'ların çalışması için workflow'un sürekli yayında kalması gerekir
7. Zamanlanmış tetikleyiciyi (schedule trigger) ihtiyacınıza göre ayarlayın veya devre dışı bırakıp manuel/Telegram tetikleme kullanın

## Notlar

- GDELT sorguları arasında rate limit nedeniyle bekleme süresi vardır; tam koşu birkaç dakika sürebilir
- Reddit ayağı olmadan da skor hesaplanır (kaynak + trend puanı normalize edilir)
- Bu repo yalnızca workflow tanımını (JSON export) içerir; credential değerleri repoya dahil değildir
