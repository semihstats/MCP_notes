# GitHub Actions Monitoring and CI/CD Analysis Tools

Bu doküman, GitHub Actions webhook'larından gelen olayları takip etmek ve CI/CD süreçlerini analiz etmek için kullanılan araçların implementasyonunu açıklar.

## Giriş

GitHub Actions iş akışlarının takip edilmesi ve sorun giderme süreçlerinin otomatikleştirilmesi için geliştirilmiş bir araç setidir. Bu araçlar, webhook'lardan gelen olayları izler, workflow durumlarını kontrol eder ve geliştiricilere detaylı raporlar sunar.

## 1. Temel Yapılandırma

### Event Dosyası Yolu

GitHub webhook'larından gelen olayların kaydedileceği dosya yolunun belirlenmesi:

```python
# File where webhook server stores events
EVENTS_FILE = Path(__file__).parent / "github_events.json"
```

Bu yapılandırma, webhook sunucusunun olayları JSON formatında saklayacağı dosyanın konumunu belirler.

## 2. Araç Implementasyonları

### 2.1 get_recent_actions_events

Son GitHub Actions olaylarını almak için kullanılan araç:

```python
@mcp.tool()
async def get_recent_actions_events(limit: int = 10) -> str:
    """Get recent GitHub Actions events received via webhook.
    Args:
        limit: Maximum number of events to return (default: 10)
    """
    # TODO: Implement this function
    # 1. Check if EVENTS_FILE exists
    # 2. Read the JSON file
    # 3. Return the most recent events (up to limit)
    # 4. Return empty list if file doesn't exist
    return json.dumps({"message": "TODO: Implement get_recent_actions_events"})
```

**Implementasyon Adımları:**

1. **Dosya Kontrolü**: Event dosyasının varlığını kontrol et
   ```python
   if not EVENTS_FILE.exists():
       return json.dumps([])
   ```

2. **Event Okuma**: Dosyayı aç ve tüm olayları yükle
   ```python
   with open(EVENTS_FILE, 'r') as f:
       events = json.load(f)
   ```

3. **Sonuç Dönüşü**: Belirlenen limit kadar son eventi döndür
   - `-limit:` parametresi ile son eventleri al
   - `indent=2` ile okunabilir JSON formatı sağla

### 2.2 get_workflow_status

GitHub Actions workflow durumlarını kontrol eden araç:

```python
@mcp.tool()
async def get_workflow_status(workflow_name: Optional[str] = None) -> str:
    """Get the current status of GitHub Actions workflows.
    Args:
        workflow_name: Optional specific workflow name to filter by
    """
    # TODO: Implement this function
    # 1. Read events from EVENTS_FILE
    # 2. Filter events for workflow_run events
    # 3. If workflow_name provided, filter by that name
    # 4. Group by workflow and show latest status
    # 5. Return formatted workflow status information
    return json.dumps({"message": "TODO: Implement get_workflow_status"})
```

**Implementasyon Adımları:**

1. **Dosya Kontrolü**: Event dosyasının varlığını kontrol et
   ```python
   if not EVENTS_FILE.exists():
       return json.dumps({"message": "No GitHub Actions events received yet"})
   ```

2. **Event Okuma**: Dosyayı yükle ve boş dosya kontrolü yap
   ```python
   with open(EVENTS_FILE, 'r') as f:
       events = json.load(f)
   
   if not events:
       return json.dumps({"message": "No GitHub Actions events received yet"})
   ```

3. **Workflow Filtering**: Sadece workflow_run eventlerini filtrele
   ```python
   workflow_events = [
       e for e in events
       if e.get("workflow_run") is not None
   ]
   ```

4. **İsim Filtreleme**: Belirli workflow ismi verilmişse filtrele
   ```python
   if workflow_name:
       workflow_events = [
           e for e in workflow_events
           if e["workflow_run"].get("name") == workflow_name
       ]
   ```
if bloğundan geçildiyse [Workflow Events örneği](Workflow_Events_örnek.md) incelenebilir.
5. **Son Durum Alma**: Her workflow için en güncel durumu al
   ```python
   workflows = {}
   for event in workflow_events:
       run = event["workflow_run"]
       name = run["name"]
       if name not in workflows or run["updated_at"] > workflows[name]["updated_at"]:
           workflows[name] = {
               "name": name,
               "status": run["status"],
               "conclusion": run.get("conclusion"),
               "run_number": run["run_number"],
               "updated_at": run["updated_at"],
               "html_url": run["html_url"]
           }
   ```

## 3. Prompt Implementasyonları

### 3.1 analyze_ci_results

CI/CD sonuçlarını analiz eden prompt:

```python
@mcp.prompt()
async def analyze_ci_results():
    """Analyze recent CI/CD results and provide insights."""
    return """Please analyze the recent CI/CD results from GitHub Actions:
  
1. First, call get_recent_actions_events() to fetch the latest CI/CD events
2. Then call get_workflow_status() to check current workflow states
3. Identify any failures or issues that need attention
4. Provide actionable next steps based on the results
  
Format your response as:
## CI/CD Status Summary
- **Overall Health**: [Good/Warning/Critical]
- **Failed Workflows**: [List any failures with links]
- **Successful Workflows**: [List recent successes]
- **Recommendations**: [Specific actions to take]
- **Trends**: [Any patterns you notice]"""
```

### 3.2 create_deployment_summary

Deployment özeti oluşturan prompt:

```python
@mcp.prompt()
async def create_deployment_summary():
    """Generate a deployment summary for team communication."""
    return """Create a deployment summary for team communication:
  
1. Check workflow status with get_workflow_status()
2. Look specifically for deployment-related workflows
3. Note the deployment outcome, timing, and any issues
  
Format as a concise message suitable for Slack:
  
🚀 **Deployment Update**
- **Status**: [✅ Success / ❌ Failed / ⏳ In Progress]
- **Environment**: [Production/Staging/Dev]
- **Version/Commit**: [If available from workflow data]
- **Duration**: [If available]
- **Key Changes**: [Brief summary if available]
- **Issues**: [Any problems encountered]
- **Next Steps**: [Required actions if failed]
  
Keep it brief but informative for team awareness."""
```

### 3.3 generate_pr_status_report

Pull Request durum raporu oluşturan prompt:

```python
@mcp.prompt()
async def generate_pr_status_report():
    """Generate a comprehensive PR status report including CI/CD results."""
    return """Generate a comprehensive PR status report:
  
1. Use analyze_file_changes() to understand what changed
2. Use get_workflow_status() to check CI/CD status
3. Use suggest_template() to recommend the appropriate PR template
4. Combine all information into a cohesive report
  
Create a detailed report with:
  
## 📋 PR Status Report
  
### 📝 Code Changes
- **Files Modified**: [Count by type - .py, .js, etc.]
- **Change Type**: [Feature/Bug/Refactor/etc.]
- **Impact Assessment**: [High/Medium/Low with reasoning]
- **Key Changes**: [Bullet points of main modifications]
  
### 🔄 CI/CD Status
- **All Checks**: [✅ Passing / ❌ Failing / ⏳ Running]
- **Test Results**: [Pass rate, failed tests if any]
- **Build Status**: [Success/Failed with details]
- **Code Quality**: [Linting, coverage if available]
  
### 📌 Recommendations
- **PR Template**: [Suggested template and why]
- **Next Steps**: [What needs to happen before merge]
- **Reviewers**: [Suggested reviewers based on files changed]
  
### ⚠️ Risks & Considerations
- [Any deployment risks]
- [Breaking changes]
- [Dependencies affected]"""
```

### 3.4 troubleshoot_workflow_failure

Workflow hata giderme prompt'u:

```python
@mcp.prompt()
async def troubleshoot_workflow_failure():
    """Help troubleshoot a failing GitHub Actions workflow."""
    return """Help troubleshoot failing GitHub Actions workflows:
  
1. Use get_recent_actions_events() to find recent failures
2. Use get_workflow_status() to see which workflows are failing
3. Analyze the failure patterns and timing
4. Provide systematic troubleshooting steps
  
Structure your response as:
  
## 🔧 Workflow Troubleshooting Guide
  
### ❌ Failed Workflow Details
- **Workflow Name**: [Name of failing workflow]
- **Failure Type**: [Test/Build/Deploy/Lint]
- **First Failed**: [When did it start failing]
- **Failure Rate**: [Intermittent or consistent]
  
### 🔍 Diagnostic Information
- **Error Patterns**: [Common error messages or symptoms]
- **Recent Changes**: [What changed before failures started]
- **Dependencies**: [External services or resources involved]
  
### 💡 Possible Causes (ordered by likelihood)
1. **[Most Likely]**: [Description and why]
2. **[Likely]**: [Description and why]
3. **[Possible]**: [Description and why]
  
### ✅ Suggested Fixes
**Immediate Actions:**
- [ ] [Quick fix to try first]
- [ ] [Second quick fix]
  
**Investigation Steps:**
- [ ] [How to gather more info]
- [ ] [Logs or data to check]
  
**Long-term Solutions:**
- [ ] [Preventive measure]
- [ ] [Process improvement]
  
### 📚 Resources
- [Relevant documentation links]
- [Similar issues or solutions]"""
```

## Sonuç

Bu araç seti, GitHub Actions workflow'larının izlenmesi, CI/CD süreçlerinin analizi ve sorun giderme işlemlerinin otomatikleştirilmesi için kapsamlı bir çözüm sunar. Webhook eventlerinden workflow durumuna kadar tüm süreçleri kapsayan bu implementasyon, geliştiricilere detaylı görünürlük ve etkili sorun giderme yetenekleri sağlar.
