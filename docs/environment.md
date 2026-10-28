# Development Environment

> Record exact versions here on the day you set up. Never assume "latest".
> Update this file whenever you upgrade a tool.

---

## Recorded: 2026-10-28

## Core Runtime

| Tool | Version | How to Verify |
|------|---------|--------------|
| OS | Windows 11 | `winver` |
| JDK | OpenJDK 17.0.18 Temurin | `java -version` |
| Maven | _fill in after install_ | `mvn -version` |
| Git | 2.55.0.windows.3 | `git --version` |
| Docker Engine | 27.5.1 | `docker --version` |
| Docker Compose | _fill in_ | `docker compose version` |

## IDE & Tools

| Tool | Version |
|------|---------|
| IntelliJ IDEA | _fill in_ |
| Postman / Bruno | _fill in_ |

---

## Maven Installation (Windows)

### Option A — Manual (Recommended)

1. Download from: https://maven.apache.org/download.cgi
   - File: `apache-maven-3.9.x-bin.zip`
2. Extract to: `C:\tools\apache-maven-3.9.x\`
3. Add to `PATH`:
   - Open **System Properties** → **Environment Variables**
   - Under **User variables**, find `Path` → Edit → New
   - Add: `C:\tools\apache-maven-3.9.x\bin`
4. Open a **new** terminal and verify:
   ```
   mvn -version
   ```
   Expected output:
   ```
   Apache Maven 3.9.x (...)
   Java version: 17.0.18, vendor: Eclipse Adoptium
   ```

### Option B — Scoop (PowerShell)

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
scoop install main/maven
```

---

## Services (Added Progressively — Do Not Fill All at Once)

> Add a row here when you first bring up a service in Docker Compose.
> Pin the exact image version you are running — never use `latest`.

| Service | Docker Image | Version Pinned | Config Location |
|---------|-------------|----------------|----------------|
| MySQL | `mysql` | _e.g. 8.0.39_ | `infra/mysql/` |
| Redis | `redis` | _e.g. 7.4.1_ | `infra/redis/` |
| Kafka | `apache/kafka` | _e.g. 3.8.0_ | `infra/kafka/` |
| MinIO | `minio/minio` | _e.g. RELEASE.2024-10-02..._ | `infra/minio/` |
| HAProxy | `haproxy` | _e.g. 3.0.5_ | `infra/haproxy/` |

---

## JAVA_HOME Setup (If Needed)

If `mvn` cannot find Java, set `JAVA_HOME`:

**PowerShell (temporary):**
```powershell
$env:JAVA_HOME = "C:\path\to\jdk-17"
```

**Permanent (User variable):**
- Name: `JAVA_HOME`
- Value: `C:\Program Files\Eclipse Adoptium\jdk-17.0.18.8-hotspot` _(adjust to your path)_

---

## Known Constraints

- All services run locally; no cloud accounts required.
- Kubernetes lab uses `kind` or `Minikube` — not a managed cluster.
- HA lab uses local VMs (Hyper-V or VirtualBox) for Keepalived — optional.
- Load tests run against local environment; results are not production benchmarks.

---

## Useful Aliases (Optional)

Add to your PowerShell profile (`$PROFILE`) for convenience:

```powershell
function oms-up   { docker compose -f D:\backend-lab-oms\compose.yaml up -d }
function oms-down { docker compose -f D:\backend-lab-oms\compose.yaml down }
function oms-logs { docker compose -f D:\backend-lab-oms\compose.yaml logs -f }
```
