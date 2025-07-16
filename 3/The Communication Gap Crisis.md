# The Communication Gap Crisis

Bu modül, geliştirici ekiplerinin birbiriyle etkili iletişim kurmasını sağlamak için tasarlanmış Slack entegrasyonu çözümüdür. Amacımız, tüm geliştiricilerin kritik güncellemeler hakkında anlık bilgi sahibi olmalarını sağlamaktır.

## Özellikler

- Otomatik Slack bildirimleri
- CI/CD hata uyarıları
- Başarılı deployment bildirimleri
- Formatlanmış mesaj şablonları

## Kurulum ve Yapılandırma

### Ortam Değişkenleri

Bu modülün çalışması için `SLACK_WEBHOOK_URL` ortam değişkeninin tanımlanması gerekir.

## Fonksiyonlar

### `send_slack_notification`

Asenkron olarak çalışan ana bildirim fonksiyonu. Tip ipuçları ve detaylı dokümantasyonla birlikte gelir.

**Parametreler:**
- `message` (str): Slack'e gönderilecek mesaj (Slack markdown formatını destekler)

**Başlangıç Şablonu:**  
```python
@mcp.tool()
async def send_slack_notification(message: str) -> str:
    """Send a formatted notification to the team Slack channel.
    
    Args:
        message: The message to send to Slack (supports Slack markdown)
    """
    webhook_url = os.getenv("SLACK_WEBHOOK_URL")
    if not webhook_url:
        return "Error: SLACK_WEBHOOK_URL environment variable not set"
    try:
        # TODO: Import requests library
        # TODO: Send POST request to webhook_url with JSON payload
        # TODO: Include the message in the JSON data
        # TODO: Handle the response and return appropriate status
        # For now, return a placeholder
        return f"TODO: Implement Slack webhook POST request for message: {message[:50]}..."
    except Exception as e:
        return f"Error sending message: {str(e)}"
```

### Ortam Değişkeni Kontrolü

Fonksiyon ilk olarak `os.getenv()` metodu ile `SLACK_WEBHOOK_URL` ortam değişkenini kontrol eder:

```python
webhook_url = os.getenv("SLACK_WEBHOOK_URL")
if not webhook_url:
    return "Error: SLACK_WEBHOOK_URL environment variable not set"
```

Bu kontrol, webhook URL'sinin tanımlanmadığı durumlarda uygun hata mesajı döner.

## Fonksiyon İmplementasyonu

### Try/Except Blok Yapısı

Fonksiyon, hata yönetimi için kapsamlı bir try/except yapısı kullanır:

#### Try Bloğu

**Payload Oluşturma:**  
HTTP isteğinde gönderilecek ana veri (payload) hazırlanır:
```python
payload = {
    "text": message,
    "mrkdwn": True
}
```

> 💡 **Not:** `mrkdwn: True` ayarı, Slack'in markdown formatını kullanmamızı sağlar.

**HTTP POST İsteği:**  
Python'ın `requests` kütüphanesi ile POST isteği gönderilir:

```python
response = requests.post(
    webhook_url,
    json=payload,
    timeout=10
)
```

**Parametreler:**
- `webhook_url`: Hedef Slack webhook URL'si
- `json=payload`: Sözlükten JSON formatına otomatik dönüşüm
- `timeout=10`: Maksimum 10 saniye bekleme süresi

**Yanıt Kontrolü:**
```python
if response.status_code == 200:
    return "✅ Message sent successfully to Slack"
else:
    return f"❌ Failed to send message. Status: {response.status_code}, Response: {response.text}"
```

#### Except Blokları

Fonksiyon üç farklı hata türünü yönetir:

1. **Zaman Aşımı Hatası:**
   ```python
   except requests.exceptions.Timeout:
       return "❌ Request timed out. Check your internet connection and try again."
   ```

2. **Bağlantı Hatası:**
   ```python
   except requests.exceptions.ConnectionError:
       return "❌ Connection error. Check your internet connection and webhook URL."
   ```

3. **Genel Hatalar:**
   ```python
   except Exception as e:
       return f"❌ Error sending message: {str(e)}"
   ```

**Tam İmplementasyon:**

```python
try:
    # Prepare the payload with proper Slack formatting
    payload = {
        "text": message,
        "mrkdwn": True
    }
    
    # Send POST request to Slack webhook
    response = requests.post(
        webhook_url,
        json=payload,
        timeout=10
    )
    
    # Check if request was successful
    if response.status_code == 200:
        return "✅ Message sent successfully to Slack"
    else:
        return f"❌ Failed to send message. Status: {response.status_code}, Response: {response.text}"
        
except requests.exceptions.Timeout:
    return "❌ Request timed out. Check your internet connection and try again."
except requests.exceptions.ConnectionError:
    return "❌ Connection error. Check your internet connection and webhook URL."
except Exception as e:
    return f"❌ Error sending message: {str(e)}"
```

## Mesaj Şablonları

### `format_ci_failure_alert`

CI/CD hatalarını bildirmek için kullanılan profesyonel Slack mesaj şablonu.

**Örnek Çıktı:**
```
🚨 *CI Failure Alert* 🚨

A CI workflow has failed:
*Workflow*: Build and Test
*Branch*: main
*Status*: Failed
*View Details*: <https://github.com/myorg/myproject/actions/runs/456|View Logs>

Please check the logs and address any issues.
```

**Prompt Tanımı:**  
```python
@mcp.prompt()
async def format_ci_failure_alert():
    """Create a Slack alert for CI/CD failures with rich formatting."""
    return """Format this GitHub Actions failure as a Slack message using ONLY Slack markdown syntax:
  
:rotating_light: *CI Failure Alert* :rotating_light:
  
A CI workflow has failed:
*Workflow*: workflow_name
*Branch*: branch_name
*Status*: Failed
*View Details*: <https://github.com/test/repo/actions/runs/123|View Logs>
  
Please check the logs and address any issues.
  
CRITICAL: Use EXACT Slack link format: <https://full-url|Link Text>
Examples:
- CORRECT: <https://github.com/user/repo|Repository>
- WRONG: [Repository](https://github.com/user/repo)
- WRONG: https://github.com/user/repo
  
Slack formatting rules:
- *text* for bold (NOT **text**)
- `text` for code
- > text for quotes
- Use simple bullet format without special characters
- :emoji_name: for emojis"""
```

### `format_ci_success_summary`

Başarılı CI/CD işlemlerini bildirmek için kullanılan kutlama mesajı şablonu.

**Örnek Çıktı:**
```
✅ *Deployment Successful* ✅

Deployment completed successfully for MyProject

*Changes:*
- Added user authentication
- Fixed payment processing bug
- Updated documentation

*Links:*
<https://github.com/myorg/myproject|View Changes>
```

**Prompt Tanımı:**
```python
@mcp.prompt()
async def format_ci_success_summary():
    """Create a Slack message celebrating successful deployments."""
    return """Format this successful GitHub Actions run as a Slack message using ONLY Slack markdown syntax:
  
:white_check_mark: *Deployment Successful* :white_check_mark:
  
Deployment completed successfully for [Repository Name]
  
*Changes:*
- Key feature or fix 1
- Key feature or fix 2
  
*Links:*
<https://github.com/user/repo|View Changes>
  
CRITICAL: Use EXACT Slack link format: <https://full-url|Link Text>
Examples:
- CORRECT: <https://github.com/user/repo|Repository>
- WRONG: [Repository](https://github.com/user/repo)
- WRONG: https://github.com/user/repo
  
Slack formatting rules:
- *text* for bold (NOT **text**)
- `text` for code
- > text for quotes
- Use simple bullet format with - or *
- :emoji_name: for emojis"""
```

## Slack Markdown Kuralları

### Önemli Formatlar

| Kullanım | Doğru Format | Yanlış Format |
|----------|--------------|---------------|
| **Kalın Metin** | `*text*` | `**text**` |
| **Kod** | `` `code` `` | `'code'` |
| **Alıntı** | `> text` | `"text"` |
| **Link** | `<https://url\|Text>` | `[Text](https://url)` |
| **Emoji** | `:emoji_name:` | `🚨` |

### Kritik Noktalar

> ⚠️ **Uyarı:** Slack link formatı GitHub Markdown'dan farklıdır. Mutlaka `<URL|Text>` formatını kullanın.

**Doğru Link Örnekleri:**
- `<https://github.com/user/repo|Repository>`
- `<https://github.com/user/repo/actions/runs/123|View Logs>`

**Yanlış Link Örnekleri:**
- `[Repository](https://github.com/user/repo)` ❌
- `https://github.com/user/repo` ❌

## Sonuç

Bu modül, geliştirici ekiplerinin CI/CD süreçlerinde karşılaştıkları iletişim sorunlarını çözmek için tasarlanmıştır. Slack entegrasyonu sayesinde:

- ✅ Anlık hata bildirimleri
- ✅ Başarılı deployment kutlamaları  
- ✅ Formatlanmış ve okunabilir mesajlar
- ✅ Güvenilir hata yönetimi

sağlanmış olur.
