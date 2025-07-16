## JSON
JavaScript Object Notation (JSON), verileri metin (string) formatında ifade etmenin basit bir yoludur. Bir veri formatıdır.
``` json
{
  "name": "Semih",
  "age": 23
}
```
## JSON-RPC
JSON (JavaScript Object Notation) + RPC (Remote Procedure Call) bu format sayesinde yazdığımız fonksiyon uzaktan çalıştırılabiliyor. Önce JSON-RPC mesajı var (kavramsal) sonra bu mesaj JSON formatına dönüştürülüyor (encode edilme kısmı).
JSON-RPC->Mektup yazma kuralları, JSON->Türkçe dili. Gibi bir benzetme yapılabilir.
## **JSON-RPC Nasıl Çalışır?**
1- İstek (Request)
``` json
{
  "jsonrpc": "2.0",        // Protokol versiyonu
  "id": 123,               // İstek numarası (takip için)
  "method": "hesapla",     // Çağrılacak fonksiyon adı
  "params": [5, 3]         // Fonksiyona gönderilecek parametreler
}
```
2- Cevap (Response)
``` json
{
  "jsonrpc": "2.0",
  "id": 123,               // Aynı istek numarası
  "result": 8              // Fonksiyonun sonucu
}
}
```
3- Hata durumu
``` json
{
  "jsonrpc": "2.0",
  "id": 123,
  "error": {
    "code": -32600,
    "message": "Geçersiz istek"
  }
}
```
MCP içerisinde açıklaması:
### Client->Server
``` json
{
  "jsonrpc": "2.0",
  "id": 42,
  "method": "get_users",
  "params": {"limit": 10}
}
```
### Server->Client
``` json
{
  "jsonrpc": "2.0",
  "id": 42,
  "result": [
    {"id": 1, "name": "Ahmet"},
    {"id": 2, "name": "Ayşe"}
  ]
}
```
