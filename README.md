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
| `REDDIT_CLIENT_ID` / `REDDIT_CLIENT_SECRET` | Sosyal skor için Reddit OAuth (opsiyonel) | Hayır |
| `GENERIC_TIMEZONE` | Zamanlanmış tetikleyici için (örn. `Europe/Istanbul`) | Önerilir |

> GDELT ve haber kaynağı RSS feed'leri (BBC/Guardian/AlJazeera/AA, Google Trends RSS) için API key gerekmez.

n8n'de env değişkenlerine node içinden erişebilmek için `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` ayarının açık olması gerekir.

## Kurulum adımları

1. n8n arayüzünde **Workflows → Import from File** ile `Jeopolitik-Haber-n8n-WORKFLOW.json` dosyasını içe aktarın
2. Yukarıdaki ortam değişkenlerini tanımlayın
3. Telegram bot webhook'unu ayarlayın (`allowed_updates` içinde `message` ve `callback_query` olmalı)
4. Workflow'u **yayına alın (publish/active)** — production webhook'ların çalışması için workflow'un sürekli yayında kalması gerekir
5. Zamanlanmış tetikleyiciyi (schedule trigger) ihtiyacınıza göre ayarlayın veya devre dışı bırakıp manuel/Telegram tetikleme kullanın

## Notlar

- GDELT sorguları arasında rate limit nedeniyle bekleme süresi vardır; tam koşu birkaç dakika sürebilir
- Reddit ayağı olmadan da skor hesaplanır (kaynak + trend puanı normalize edilir)
- Bu repo yalnızca workflow tanımını (JSON export) içerir; credential değerleri repoya dahil değildir
