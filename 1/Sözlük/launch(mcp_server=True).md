`.lunch` metodunda `mcp_server=True` girildiğinde:
1- Gradio araçları otomatik olarak MCP araçlarına dönüşüyor.
2- Input bileşenleri, tool argümanlarının şemasına dönüşür.
3- Output bileşenleri cevap formatını belirler.
4- Gradio sunucusu artık MCP protokol mesajlarını da dinliyor. (Sunucu aynı port üzerinde her iki tür (HTTP->GET,POST/MCP->JSON-RPC) isteği de karşılar MCP istekleri /mcp endpoint'ine yönlendirilir.)
5- MCP'nin iletişim protokolü tanımlanır. Bu işlem HTTP+SSE üzerinden JSON-RPC ile yapılır.