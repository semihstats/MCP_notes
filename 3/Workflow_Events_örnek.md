```Python
workflow_events = [
    {
        "workflow_run": {
            "name": "CI/CD Pipeline",
            "status": "completed",
            "conclusion": "success", 
            "run_number": 42,
            "updated_at": "2024-01-01T10:30:00Z",  # ← Eski tarih
            "html_url": "https://github.com/user/repo/actions/runs/123"
        }
    },
    {
        "workflow_run": {
            "name": "Security Scan",
            "status": "in_progress",
            "conclusion": null,
            "run_number": 15, 
            "updated_at": "2024-01-01T10:45:00Z",
            "html_url": "https://github.com/user/repo/actions/runs/124"
        }
    },
    {
        "workflow_run": {
            "name": "CI/CD Pipeline",
            "status": "completed",
            "conclusion": "failure",
            "run_number": 43,
            "updated_at": "2024-01-01T11:00:00Z",  # ← Yeni tarih
            "html_url": "https://github.com/user/repo/actions/runs/125"
        }
    }
]
```  
Örneğin yukarıdaki durumda iki tane "CI/CD Pipeline" isimli run var. Bu iki durumdan sonuncu workflows sözlüğünün name anahtarına aşağıdaki kısım atanır:  
```Python
{
    "name": "CI/CD Pipeline",
    "status": "completed", 
    "conclusion": "failure",         # ← En son durumu
    "run_number": 43,                # ← En son run
    "updated_at": "2024-01-01T11:00:00Z",
    "html_url": "https://github.com/user/repo/actions/runs/125"
}
```
workflow_events içerisinde bir de daha önce görülmemiş bir run var. Bu olduğu gibi alınır:
```Python
{
	"workflow_run": {
	"name": "Security Scan",
	"status": "in_progress",
	"conclusion": null,
    "run_number": 15, 
	"updated_at": "2024-01-01T10:45:00Z",
	"html_url": "https://github.com/user/repo/actions/runs/124"
}
```  
Tüm bu işlemler ile workflows.values() Sonucu:  
```Python
[
    {
        "name": "CI/CD Pipeline",
        "status": "completed", 
        "conclusion": "failure",         # ← En son durumu
        "run_number": 43,                # ← En son run
        "updated_at": "2024-01-01T11:00:00Z",
        "html_url": "https://github.com/user/repo/actions/runs/125"
    },
    {
        "name": "Security Scan",
        "status": "in_progress",
        "conclusion": null,
        "run_number": 15,
        "updated_at": "2024-01-01T10:45:00Z", 
        "html_url": "https://github.com/user/repo/actions/runs/124"
    }
]
```
