# Model Context Protocol (MCP) - Advanced Development  
**Building Custom Workflow Servers for Claude Code**


---

Bu bölümde **PR (Pull Request) Agent Workflow Server** geliştirilecek. Temel amacı, kod geliştirme sürecini otomatikleştirmek ve akıllı hale getirmek.

### Temel Bileşenler

1. **Smart PR Management**  
   - Kod değişiklerini analiz eder  
   - Değişikliğe uygun PR template seçer  

2. **CI/CD Monitoring**  
   - GitHub Actions'ları izler  
   - **Standardized Prompts** (MCP'nin bir özelliği) kullanarak raporlama yapar  

3. **Team Communication**  
   - **Slack entegrasyonu** ile takım üyelerini bilgilendirir  

**Ön Gereklilikler:**
- Claude Code'un [kurulumu](https://docs.anthropic.com/en/docs/claude-code/setup)  

---

Bu modülde aşağıdaki araçlar geliştirilecek:

- **`analyze_file_changes`** - Değişiklikleri görmek için dosya analizi  
- **`get_pr_templates`** - Claude'nin seçebileceği 7 template gösterir  
- **`suggest_template`** - Claude 7 seçenekten birini seçer  

### Ek Özellikler

- Değişiklik özeti yazma  
- Güvenlik sorunu tespiti  
- Sonrasında yapılacaklar listesi oluşturma  
- Önceliklendirme  
#### Ön Gereklilikler
- Git kurulumu  
- uv paket yöneticisi kurulumu  

#### Başlangıç Kodu

```bash
# Repo klonlama
git clone https://github.com/huggingface/mcp-course.git

# Proje dizinine geçiş
cd mcp-course/projects/unit3/build-mcp-server/starter

# Sanal ortam oluşturma
uv venv .venv
.venv\Scripts\activate
```

### Görev

`server.py` dosyasında belirtilen 3 aracı geliştirmeniz gerekmektedir.

> **Çözüm:** Bu modülün çözümü [[The PR Chaos at CodeCraft Studios]] dosyasında bulunabilir.

---

## Modül 2: GitHub Actions Entegrasyonu

### Ne Geliştirilecek

- **Webhook server** - GitHub Actions erişimi için  
- **Yeni araçlar** - CI/CD durumunu görüntülemek için  
- **MCP Prompts** - Standardize edilmiş prompt şablonları  
- **Gerçek zamanlı erişim** - GitHub repolarına anlık bağlantı  

### Ön Gereklilikler

Bu modülü tamamlamak için:
- GitHub Actions bilgisi  
- GitHub repo sahibi olma  
- **Cloudflare Tunnel (cloudflared)** kurulumu  

### Anahtar Kavramlar

- **MCP Prompts:** Hazır prompt şablonları  
- **Webhook Integration:** MCP server 2 servisi birden çalıştırır:
  - MCP server  
  - Webhook server (8080 portunda GitHub Actions bilgisi için)  

### Proje Mimarisi

```plaintext
github-actions-integration/
├── starter/          # Başlangıç noktanız
│   ├── server.py     # Module 1 kodu + TODOs
│   ├── pyproject.toml
│   └── README.md
└── solution/         # Tam uygulama
    ├── server.py     # Full webhook + prompts
    ├── pyproject.toml
    └── README.md
```

### Kurulum

```bash
# Starter klasörüne geçiş
cd github-actions-integration/starter

# Bağımlılıkları yükleme
uv sync
```

> **Çözüm:** Bu modülün kodu [[The Silent Failures Strike]] dosyasında açıklanmıştır.

#### Adım 5: Cloudflare Tunnel ile Test

1. **MCP server'ı başlatma:**
   ```bash
   uv run server.py
   ```

2. **Cloudflare Tunnel çalıştırma:** (ayrı terminalde)
   ```bash
   cloudflared tunnel --url http://localhost:8080
   ```
   > Bu komut public bir URL sağlar

3. **GitHub Webhook Konfigürasyonu** gereklidir

4. **Claude Code ile test** yapılabilir

### Alıştırmalar

Modülün sonunda 3 [alıştırma](https://huggingface.co/learn/mcp-course/unit3/github-actions-integration#exercises) bulunmaktadır.

### Yaygın Sorunlar

Karşılaşılabilecek [sorunlar](https://huggingface.co/learn/mcp-course/unit3/github-actions-integration#common-issues) için rehber mevcuttur.

---

## Slack Bildirimleri

### Proje Yapısı

```plaintext
slack-notification/
├── starter/          # Başlangıç noktanız
│   ├── server.py     # Modül 1+2 kodu + TODOs
│   ├── webhook_server.py  # Modül 2'den
│   ├── pyproject.toml
│   └── README.md
└── solution/         # Tam uygulama
    ├── server.py     # Full Slack entegrasyonu
    ├── webhook_server.py
    └── README.md
```

### Uygulama Adımları

#### 1. Slack Webhook Oluşturma

1. [Slack API](https://api.slack.com/apps) adresine gidin  
2. Hesap açın veya giriş yapın  
3. **Create new app** → **"From scratch"** seçin  
4. App adına `MCP Course Notifications` girin  
5. App'i oluşturun  
6. Sol panelden **Incoming Webhooks** sekmesine gidin ve aktive edin  
7. **Add New Webhook** seçin  
8. İstenilen kanalı seçin ve yetkilendirin  
9. Webhook URL'ini kopyalayın  

#### 2. Webhook Test Etme

Git bash kullanarak (YOUR_WEBHOOK_URL kısmını değiştirin):

```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Hello from MCP Course!"}' \
  YOUR_WEBHOOK_URL
```

#### 3. Ortam Değişkeni Ayarlama

VS Code terminalde git bash açın ve aşağıdaki kodu çalıştırın:

```bash
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
```

> **Önemli:** Bu URL güvenli tutulmalı ve kod içinde belirtilmemeli!

### Görev

`server.py` dosyasına aşağıdakileri ekleyin:

- **MCP Tool:** `send_slack_notification` - Slack bildirimi gönderen araç  
- **2 Prompt:** 
  - `format_ci_failure_alert`  
  - `format_ci_success_alert`  

> **Çözüm:** Bu işlemler [[The Communication Gap Crisis]] dosyasında açıklanmıştır.

### Tam Workflow Testi

1. **Tüm servisleri başlatın:**
   ```bash
   # Terminal 1: Webhook server
   python webhook_server.py
   
   # Terminal 2: MCP server
   uv run server.py
   
   # Terminal 3: Cloudflare Tunnel
   cloudflared tunnel --url http://localhost:8080
   ```

2. **Claude Code ile tam entegrasyonu test edin**

### Entegrasyon Doğrulama 

Çözüm klasöründeki `manual_test.md` dosyasındaki adımları takip ederek gerçek GitHub repo kullanmadan test yapabilirsiniz.

### Yaygın Sorunlar

- **Webhook URL:** Ortam değişkeni doğru ayarlanmalı  
- **Message Formatting:** Slack'in markdown formatı GitHub'dan farklıdır  
- **Network Errors:** Webhook requests sırasında bağlantı hataları olabilir  

---

### 📖 Unit 3 Çözüm Rehberi

[Proje çözüm rehberi](https://huggingface.co/learn/mcp-course/unit3/build-mcp-server-solution-walkthrough) ile gerçekleştirilen projenin detaylı açıklamaları bulunmaktadır.

Bu rehber, tüm modüllerin adım adım nasıl tamamlandığını gösterir ve best practices örnekleri sunar.

---

## Sonuç

Bu modül ile şunları başardınız:

- ✅ **MCP Server** geliştirme  
- ✅ **GitHub Actions** entegrasyonu  
- ✅ **Slack** bildirimleri  
- ✅ **Cloudflare Tunnel** kullanımı  
- ✅ **Claude Code** entegrasyonu  

### Sonraki Adımlar

- Projenizi kendi gereksinimlerinize göre özelleştirin  
- Farklı CI/CD platformları ile entegrasyon deneyin  
- Ek bildiri kanalları ekleyin  
- Gelişmiş analitik özellikler ekleyin  

