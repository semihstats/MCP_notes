Çevre değişkenleri, işletim sisteminde saklanan ve programların erişebileceği değerlerdir. Gizli bilgileri kodun içine yazmak yerine dışarıdan vermek için kullanılır.
MCP'de örnek env kullanımı:
```JSON
{
  "servers": [
    {
      "name": "GitHub API Server",
      "transport": {
        "type": "stdio",
        "command": "python",
        "args": ["github_server.py"],
        "env": {
          "GITHUB_TOKEN": "ghp_1234567890abcdef",
          "API_TIMEOUT": "30"
        }
      }
    }
  ]
}
```
Yukarıdaki gibi şifre ve API'lere erişim için aşağıdaki gibi yöntemler uygulanabilir:
```Python
# GÜVENSİZ! Token koda gömülü
github_token = "ghp_1234567890abcdef"
```
Bu kısım kod içerisinde tutulursa kötü niyeti kişilerin kullanımına açmış olursunuz. Aşağıdaki yöntem ise:
```Python
# GÜVENLİ! Token dışarıdan geliyor
github_token = os.environ.get("GITHUB_TOKEN")
```
Bu yöntemde mcp.json dosyası paylaşılmadığı müddetçe sistem gerekli token bilgisini alır ve güvenle çalışmaya devam eder.
