Được, đây là bản tổng kết chi tiết toàn bộ quá trình để bạn ghi chú lại.

MỤC ĐÍCH TỔNG THỂ

Xây dựng hệ thống quản lý cấu hình Kubernetes cho 6 microservice Java (account, payment, fraud-detection, transaction, notification, api-gateway), sao cho:

Không lặp lại code — 6 service dùng chung 1 "khuôn mẫu" (Helm chart).
Không lặp lại config theo môi trường — 3 môi trường dev/staging/prod tự động khác nhau (namespace, số bản sao, phiên bản image) chỉ bằng cách đổi 1 file cấu hình nhỏ (Kustomize overlay).
CÔNG CỤ ĐÃ CÀI
Công cụ	Vai trò
Helm	Định nghĩa "khuôn mẫu" YAML có thể tham số hóa (dùng biến thay vì hardcode)
Kustomize	Lớp "vá" (patch) đè lên khuôn mẫu đó, tùy biến theo từng môi trường

Cài qua winget install Helm.Helm và winget install Kubernetes.kustomize.

CẤU TRÚC THƯ MỤC ĐÃ TẠO
banking-system/
├── charts/
│   └── microservice/              ← Helm chart dùng chung
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── .helmignore
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── configmap.yaml
│           └── _helpers.tpl
└── k8s/
    ├── base/
    │   └── kustomization.yaml     ← gọi Helm chart cho cả 6 service
    └── overlays/
        ├── dev/
        │   └── kustomization.yaml
        ├── staging/
        │   └── kustomization.yaml
        └── prod/
            └── kustomization.yaml
Ý NGHĨA TỪNG FILE
charts/microservice/Chart.yaml

File metadata bắt buộc của Helm — khai báo tên chart, version. Được tự sinh khi chạy helm create, không cần sửa.

charts/microservice/values.yaml

File giá trị mặc định — nơi định nghĩa các biến sẽ được "đổ vào" template. Ví dụ: serviceName, image.repository, resources.requests.cpu... Đây là giá trị fallback nếu không có gì override.

charts/microservice/.helmignore

Danh sách file/thư mục Helm bỏ qua khi đóng gói chart (giống .gitignore). Tự sinh, không cần sửa.

charts/microservice/templates/deployment.yaml

Khuôn mẫu Deployment dùng chung cho cả 6 service. Thay vì viết cứng tên service, image... file này dùng biến {{ .Values.serviceName }}, {{ .Values.image.repository }} — mỗi lần render, Helm thay biến bằng giá trị thật tương ứng với từng service.

charts/microservice/templates/service.yaml

Khuôn mẫu Kubernetes Service (expose port 8080) — tương tự deployment.yaml, dùng chung, chỉ khác tên qua biến.

charts/microservice/templates/configmap.yaml

Khuôn mẫu ConfigMap chứa biến môi trường (SPRING_PROFILES_ACTIVE...) cho Spring Boot app, cũng tham số hóa qua {{ .Values.env }}.

charts/microservice/templates/_helpers.tpl

File tiện ích Helm tự sinh, chứa các hàm helper để đặt tên resource nhất quán (không dùng trực tiếp trong task này, giữ nguyên mặc định).

k8s/base/kustomization.yaml

File quan trọng nhất của tầng "base" — nơi gọi chart microservice 6 lần, mỗi lần truyền giá trị riêng (serviceName, image.repository) cho từng service cụ thể (account-service, payment-service...). Đây là bước biến 1 chart chung thành 6 bộ manifest riêng biệt.

k8s/overlays/dev/kustomization.yaml

File "công tắc" cho môi trường dev:

Trỏ về ../../base (kế thừa toàn bộ 6 service từ base).
Patch replicas: 1 (chạy ít bản sao, tiết kiệm tài nguyên).
Patch namespace: banking-dev (cách ly dev khỏi các môi trường khác).
Đổi image tag thành dev-latest.
k8s/overlays/staging/kustomization.yaml

Tương tự dev nhưng:

replicas: 2 (test khả năng chịu tải nhẹ).
namespace: banking-staging.
Image tag staging-latest.
k8s/overlays/prod/kustomization.yaml

Cấu hình cho production (khách hàng thật dùng):

replicas: 5 (chịu tải cao).
namespace: banking-prod.
Image tag cố định v1.0.0 (không dùng "latest" để tránh deploy nhầm bản chưa test).
QUY TRÌNH HOẠT ĐỘNG (LUỒNG DỮ LIỆU)
values.yaml (giá trị mặc định)
        ↓
templates/*.yaml (khuôn mẫu Helm, có biến {{ }})
        ↓ [helm render qua k8s/base/kustomization.yaml]
6 bộ Deployment/Service/ConfigMap (chưa theo môi trường)
        ↓ [Kustomize overlay patch: namespace, replicas, image tag]
18 manifest YAML hoàn chỉnh, đúng theo dev/staging/prod
        ↓
kubectl apply -f - (đưa lên cluster thật)
LỆNH KIỂM TRA ĐÃ DÙNG XUYÊN SUỐT
bash
# Xem thử kết quả render (KHÔNG deploy thật)
kustomize build --enable-helm --load-restrictor LoadRestrictionsNone k8s/overlays/dev

--enable-helm: bật tính năng gọi Helm chart trong Kustomize (mặc định tắt).
--load-restrictor LoadRestrictionsNone: cho phép Kustomize đọc file nằm ngoài thư mục k8s/ (vì chart nằm ở charts/, khác nhánh cây thư mục).

BƯỚC ĐANG LÀM DỞ: COMMIT LÊN GIT
bash
git add charts/ k8s/          # chỉ thêm 2 thư mục mới, bỏ qua .idea/
git commit -m "Add Helm chart and Kustomize overlays for dev/staging/prod environments"
git push origin developer     # đẩy lên nhánh developer trên remote

Ý nghĩa: lưu toàn bộ cấu hình vào lịch sử Git, đồng đội khác pull về là có ngay bộ chart + overlay để deploy, không cần làm lại từ đầu.


-----------

kubectl kustomize k8s\base\ --enable-helm --load-restrictor LoadRestrictionsNone

----
 tree k8s /F
 
  kubectl kustomize k8s\base --enable-helm --load-restrictor LoadRestrictionsNone
  
  -----
  
  kubectl apply -k k8s\base --enable-helm --load-restrictor LoadRestrictionsNone
  
  ---
  
  kubectl kustomize k8s\base --enable-helm --load-restrictor LoadRestrictionsNone > banking-rendered.yaml
  
  --
 kubectl apply -f .\banking-rendered.yaml
 
 --
 
 kubectl get pods
 
 ---
 
 PS D:\antn\Admin\Java-Banking-Project\banking-system> kubectl get pods
NAME                                       READY   STATUS              RESTARTS   AGE
account-service-68f47cd6cb-m54mp           0/1     ErrImagePull        0          40s
api-gateway-76854cdbc8-pfklp               0/1     ImagePullBackOff    0          40s
fraud-detection-service-589cfdf7f4-dw7r5   0/1     ErrImagePull        0          40s
notification-service-74df6b8568-mtknt      0/1     ContainerCreating   0          40s
payment-service-5fd89ccc98-qjjm4           0/1     ContainerCreating   0          40s
transaction-service-7cd77bf5db-mgq6v       0/1     ContainerCreating   0          40s
PS D:\antn\Admin\Java-Banking-Project\banking-system>


---------

kubectl describe pod account-service-68f47cd6cb-m54mp


kubectl apply -f banking-rendered.yaml


kubectl logs account-service-774f77dbcf-d69nk -n banking-dev --tail=50

--------------

# Incident Postmortem: banking-dev Pods Failing After Docker Desktop Restart

## Summary

After restarting Docker Desktop, `kubectl get pods -n banking-dev` no longer showed 9/9 pods `Running`. Six of the nine services (all custom-built Java microservices) went into `ImagePullBackOff` / `ErrImagePull`. The three infrastructure pods pulled from the local registry mirror (`kafka`, `mysql`, `zookeeper`) and `redis` (pulled from Docker Hub) were unaffected.

## Symptom

```
kubectl get pods -n banking-dev
```

```
account-service-...           0/1   ImagePullBackOff
api-gateway-...                0/1   ImagePullBackOff
fraud-detection-service-...    0/1   ErrImagePull
notification-service-...       0/1   ErrImagePull
payment-service-...            0/1   ImagePullBackOff
transaction-service-...        0/1   ErrImagePull
kafka-...                      1/1   Running
mysql-...                      1/1   Running
redis-...                      1/1   Running
zookeeper-...                  1/1   Running
```

## Diagnostic Steps

### 1. Inspect pod events

```
kubectl describe pod <pod-name> -n banking-dev
```

Relevant event:

```
Failed to pull image "your-registry/account-service:latest":
failed to resolve reference "docker.io/your-registry/account-service:latest":
pull access denied, repository does not exist or may require authorization
```

This showed the pod was trying to pull `your-registry/account-service:latest` — a name that resolves to Docker Hub (`docker.io/your-registry/...`), not to any real registry.

### 2. Confirm the local registry itself was healthy

```
docker ps | findstr registry
```

Result: the `registry:2` container was up and listening on `0.0.0.0:5000`. This ruled out "registry container is down" as the cause.

### 3. Compare the image actually built/pushed vs. the image referenced by the pod

```
docker images | findstr localhost:5000
```

Result: images were correctly tagged and present as `localhost:5000/<service>:dev-latest`. So the real, pushed image used the `localhost:5000` host and the `dev-latest` tag — neither of which matched what the pod was requesting (`your-registry/...:latest`).

### 4. Proof the root cause was NOT a bug in the k8s/ manifest files

This was the critical step — confirming the Kustomize configuration itself was correct, and the problem was an operational/process issue (how manifests were generated and applied), not a file content bug.

**a. Confirm the dev overlay already declares the correct image mapping:**

```
type k8s\overlays\dev\kustomization.yaml
```

The file already contained an `images:` transformer:

```yaml
images:
  - name: your-registry/account-service
    newName: localhost:5000/account-service
    newTag: dev-latest
  # ...same pattern for the other 5 services
```

This confirms the overlay, if used to render, would correctly rewrite `your-registry/*` to `localhost:5000/*:dev-latest`.

**b. Render directly from the dev overlay and inspect the output:**

```
kubectl kustomize k8s\overlays\dev --enable-helm --load-restrictor LoadRestrictionsNone > k8s\overlays\dev\banking-rendered.yaml
Select-String "image:" k8s\overlays\dev\banking-rendered.yaml
```

(Note: use PowerShell's `Select-String`, not `findstr` — `kubectl kustomize` output on Windows is UTF-16, and `findstr` silently misses matches in Unicode files.)

Result: every service image in the rendered output was correctly `localhost:5000/<service>:dev-latest`. This proved the overlay + transformer machinery worked exactly as designed — **the files in `k8s/` were not the bug.**

**c. Find where the placeholder `your-registry` actually originates (expected — this is normal in `base`):**

```
Select-String "your-registry" charts\microservice\values.yaml
Select-String "your-registry" k8s\base\kustomization.yaml
```

Result: `your-registry` is the intentional Helm chart default / base-layer placeholder value. It is expected to appear in `base` — it is only supposed to survive into the final manifest if `base` is applied directly instead of an environment overlay. Finding it in `base` is normal design, not a bug.

**d. Definitive proof: inspect what was actually last applied to the live cluster**

```
kubectl get deployment account-service -n banking-dev -o jsonpath="{.metadata.annotations.kubectl\.kubernetes\.io/last-applied-configuration}"
```

Result (before fix):

```json
"image":"your-registry/account-service:latest"
```

This is the smoking gun: the annotation Kubernetes stores from the actual `kubectl apply` proves that at some point, a manifest containing the raw `your-registry/*:latest` value (i.e., **not** rendered through the dev overlay's image transformer) was applied directly to the `banking-dev` namespace. This could only happen by applying from `k8s/base` (or an old/incorrect rendered file) instead of `k8s/overlays/dev`.

## Root Cause

The live deployments in `banking-dev` had been created/updated at some point by applying manifests rendered from `k8s/base` (or a stale rendered file), bypassing the `k8s/overlays/dev` Kustomize layer that rewrites `your-registry/*` → `localhost:5000/*:dev-latest`. As a result, the live Deployment objects referenced a non-existent image reference (`your-registry/<service>:latest`).

This stayed hidden because the pods had already been scheduled and their images were cached in containerd on the Docker Desktop Kubernetes node (`desktop-control-plane`), so no actual pull was needed for them to stay `Running`. When Docker Desktop was restarted, containerd's image cache was cleared, forcing kubelet to re-pull the image — which then failed because `your-registry/<service>:latest` does not exist on any real registry.

**In short:** the bug was in the *process* (applying from the wrong Kustomize layer), not in the content of any file under `k8s/`. The overlay's transformer logic was correct the whole time.

## Fix

1. Render manifests from the **dev overlay**, not `base`:
   ```
   kubectl kustomize k8s\overlays\dev --enable-helm --load-restrictor LoadRestrictionsNone > k8s\overlays\dev\banking-rendered.yaml
   ```
2. Verify the rendered output has correct images before applying:
   ```
   Select-String "image:" k8s\overlays\dev\banking-rendered.yaml
   ```
3. Apply the correctly rendered file:
   ```
   kubectl apply -f k8s\overlays\dev\banking-rendered.yaml
   ```
4. Confirm the fix stuck, by re-checking the live annotation:
   ```
   kubectl get deployment account-service -n banking-dev -o jsonpath="{.metadata.annotations.kubectl\.kubernetes\.io/last-applied-configuration}"
   ```
   Expected: `"image":"localhost:5000/account-service:dev-latest"`.

After this, `kubectl get pods -n banking-dev` returned to `9/9 Running`.

## Standing Rules to Prevent Recurrence

1. **Always render/apply from `k8s/overlays/dev`** (or `staging`/`prod` as appropriate). Never run `kubectl apply -k k8s/base` or apply a file rendered from `base` directly — `base` is an incomplete template by design.
2. **Keep the local registry container (`localhost:5000`) always running.** Check its restart policy:
   ```
   docker inspect registry --format "{{.HostConfig.RestartPolicy.Name}}"
   ```
   It should be `always` or `unless-stopped` so it survives Docker Desktop restarts.
3. **After rebuilding a service image, always re-push it to `localhost:5000/<service>:dev-latest`** before redeploying — otherwise the cluster will silently keep running the previous image version under the same tag.
4. Prefer `Select-String` over `findstr` when inspecting any file produced by `kubectl kustomize`, since that output is UTF-16 on Windows and `findstr` can silently return no matches even when the target string is present.


---------------------

Về cơ bản là có, nhưng "luôn luôn" thì mình cần nói rõ điều kiện đi kèm để bạn không bị bất ngờ nếu có lần lỡ tay:

Sẽ luôn 9/9 Running nếu 3 điều kiện này giữ nguyên:

Deployment hiện tại trên cluster vẫn giữ đúng image localhost:5000/xxx:dev-latest (đã xác nhận qua annotation last-applied-configuration) — điều này không tự đổi khi tắt/mở máy, nó chỉ đổi khi ai đó chạy kubectl apply với file sai.
Container registry localhost:5000 tự khởi động lại cùng Docker Desktop (không bị xoá hoặc quên start).
Không ai (kể cả chính bạn) vô tình kubectl apply -k k8s\base hoặc apply file rendered từ base thay vì từ overlays\dev.

Vì Docker Desktop restart chỉ xoá cache image của containerd, không xoá cấu hình Deployment đã lưu trong etcd — nên chỉ cần image tồn tại trong registry để pull lại là đủ, không cần apply lại gì cả. Việc bạn tắt/mở laptop bình thường sẽ không đụng đến 3 điều kiện trên.

Nếu muốn chắc chắn 100%, cứ tắt/mở laptop thử rồi chạy kubectl get pods -n banking-dev xem — nếu ra 9/9 thì xác nhận fix đã bền vững.


-------------

Dựa trên danh sách bạn từng muốn áp dụng (probes, HPA, PDB, NetworkPolicy, secrets, Ingress, GitOps, service mesh, observability), đây là gợi ý thứ tự triển khai theo độ ưu tiên thực tế — từ nền tảng đến nâng cao:

1. Probes (Liveness/Readiness/Startup) — nên làm đầu tiên

Vì hệ thống bạn vừa gặp sự cố pull image, probe sẽ giúp phát hiện sớm khi service "Running" nhưng thực chất chưa sẵn sàng (DB chưa connect, Kafka chưa ready...).

yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8081
  initialDelaySeconds: 30
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8081

Spring Boot Actuator đã hỗ trợ sẵn 2 endpoint này — chỉ cần bật management.endpoint.health.probes.enabled=true.

2. Resource Requests/Limits chuẩn hoá + HPA

Bạn đã có resources.limits/requests cơ bản. Bước tiếp theo là Horizontal Pod Autoscaler dựa trên CPU/memory, để tự scale khi tải tăng — rất hợp để demo kỹ năng "production-ready" trong CV/phỏng vấn.

kubectl autoscale deployment payment-service --cpu-percent=70 --min=1 --max=5 -n banking-dev
3. Secrets Management đúng chuẩn

Hiện bạn dùng Secret thô (base64, không mã hoá thật). Nâng cấp lên:

Sealed Secrets (Bitnami) — mã hoá secret để commit an toàn vào Git.
Hoặc External Secrets Operator nếu muốn tích hợp Vault/AWS Secrets Manager sau này (kỹ năng rất được đánh giá cao).
4. NetworkPolicy

Namespace banking-dev hiện chưa có ranh giới network — mọi pod gọi được lẫn nhau tự do. Thêm NetworkPolicy để giới hạn, ví dụ chỉ api-gateway mới được gọi vào các service khác, không cho FE gọi thẳng payment-service.

5. Ingress

Thay vì mỗi service tự expose port riêng, dùng Ingress Controller (nginx-ingress) để có 1 điểm vào duy nhất, route theo path/host — gần với kiến trúc thật hơn.

6. Observability (Prometheus + Grafana)

Vì bạn đã có Spring Boot Actuator, chỉ cần thêm micrometer-registry-prometheus là có metrics ngay. Đây là kỹ năng có giá trị cao khi phỏng vấn vị trí backend/DevOps.

7. PDB (PodDisruptionBudget) + GitOps (ArgoCD) — làm sau cùng

PDB chỉ thực sự có ý nghĩa khi bạn đã có nhiều replicas + HPA. GitOps (ArgoCD) là bước nâng cao nhất — tự động sync cluster theo Git, phù hợp khi bạn đã quen thuộc toàn bộ workflow thủ công ở trên.

Service mesh (Istio/Linkerd) — không ưu tiên lúc này

Với quy mô 6 service demo, service mesh sẽ overkill và làm hệ thống phức tạp không cần thiết. Nên để cuối cùng, chỉ làm nếu bạn thực sự muốn học riêng công nghệ này.

Bạn muốn bắt đầu từ Probes hay HPA trước? Mình có thể hướng dẫn implement chi tiết ngay.



-------------------

kubectl port-forward svc/api-gateway 8080:8080 -n banking-dev


------------------------------------------------------------------------------------------------------------------------------

# Banking System Crash-Loop: Follow-Ups & Full Fix Report

## Part 1 — Remaining Follow-Up Items (Not Urgent)

**1. Kafka restarted once (~8 minutes before the last check).**
This happened because MySQL, Zookeeper, and several other pods were all restarted together during the fix rollout. It is not a new or separate bug — Kafka depends on Zookeeper, and Zookeeper's pod also cycled during that same rollout. No action is needed unless Kafka keeps restarting on its own afterward.

**2. `payment-service` and `transaction-service` are still running their old container images.**
Both services already got the Kubernetes-level fix (the `wait-for-mysql` init container), which was enough to stop the crash-loop we were seeing. However, the Hikari connection-pool settings (`connection-timeout`, `initialization-fail-timeout=-1`, `max-lifetime`) were added to their `application.yaml` files but never actually built into a new Docker image and pushed to the registry — the Maven build step failed with a `.\mvnw` "command not found" error and was never resolved.

Why this still matters: the init container only protects the *very first moment* the pod starts (it waits for MySQL's port to open before the app container is even created). It does nothing if MySQL becomes briefly unreachable *after* the app has already started and been running for a while — for example, if MySQL restarts, or a network hiccup drops the connection mid-session. In that scenario, without the Hikari settings, the same "Communications link failure / Connection refused" crash could happen again, just triggered differently. Rebuilding and pushing the payment-service and transaction-service images is the way to close that remaining gap completely.

---

## Part 2 — Full Investigation and Fix Timeline

### Symptom
Three of the ten pods in the `banking-dev` namespace — `account-service`, `payment-service`, and `transaction-service` — were stuck in a restart loop, with restart counts climbing into the dozens (29, 50, 50) over roughly 12–14 hours, while every other service in the same namespace (api-gateway, fraud-detection-service, kafka, mysql, notification-service, redis, zookeeper) stayed stable with only a handful of restarts from normal cluster activity.

### Step 1 — Reading the crash logs
Running `kubectl logs <pod> --previous` on account-service showed a clear, repeatable stack trace:

```
org.hibernate.exception.JDBCConnectionException: Unable to obtain isolated JDBC connection
Caused by: com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure
Caused by: java.net.ConnectException: Connection refused
```

This happens during Spring Boot's startup sequence, specifically when Hibernate tries to open a JDBC connection to run its schema DDL check. The exact same failure repeated across account-service, payment-service, and transaction-service — all three are the only services in the deployment that talk directly to MySQL.

### Step 2 — Diagnosing the root cause
The pattern was: the Spring Boot application container starts up immediately when the pod is scheduled, but MySQL takes noticeably longer to become ready to accept TCP connections (its own container has to initialize its data directory, start a temporary server, shut it down, and start the real server — this alone takes roughly 60–90 seconds based on MySQL's own logs). HikariCP, the connection pool Spring Boot uses by default, was configured with no tolerance for this: as soon as it tried to open the very first connection and got refused, it threw an exception immediately, which caused the whole Spring application context to fail to initialize, which caused the JVM process to exit. Kubernetes then restarted the container, and the exact same race condition happened again on every retry — hence the endlessly climbing restart count.

In short: **a startup-time race condition between the application containers and the MySQL container, with no retry logic on the application side.**

### Step 3 — Designing the fix
Two complementary fixes were planned:
1. **Kubernetes-level fix:** add an `initContainer` to each of the three affected deployments. An init container is a small container that must run to completion *before* the main application container is even created. Using a minimal `busybox` image running `until nc -z -w2 mysql-service 3306; do sleep 3; done`, the pod would simply wait, checking every 3 seconds, until MySQL's port 3306 was actually open — guaranteeing the main app container never even starts until MySQL can accept connections.
2. **Application-level fix:** add HikariCP settings (`connection-timeout=30000`, `initialization-fail-timeout=-1`, `max-lifetime=600000`) to each service's `application.yaml`. The key setting is `initialization-fail-timeout=-1`, which tells Hikari not to throw an exception if it can't get a connection immediately at startup — instead it lets the Spring context finish initializing and keeps retrying the connection pool in the background. This protects against any future MySQL unavailability, not just the one at pod-startup time.

### Step 4 — First attempt and an unexpected obstacle
The Hikari settings were added to `account-service`'s `application.yaml`, and an `initContainer` block was added directly to `k8s/base/account-deployment.yaml`. The image was rebuilt, pushed to the local registry, and the Kustomize-rendered manifest was re-applied. But checking the running pod afterward showed it was still the *exact same pod* (same name, same age) — meaning nothing had actually changed on the cluster, despite the new image being pushed.

Two separate problems were uncovered here:
- The dev overlay's `kustomization.yaml` had a patch hard-coding `imagePullPolicy: IfNotPresent` on every deployment. This meant Kubernetes would keep reusing whatever image was already cached locally, rather than pulling the freshly pushed one — so simply pushing a new image was never enough to actually update the running container.
- Even after manually forcing a rollout (`kubectl rollout restart`), the new pod still crashed, and inspecting the deployment on the cluster with `kubectl get deployment account-service -o yaml` showed **no `initContainers` section at all** — the edit to `account-deployment.yaml` had not taken effect.

### Step 5 — Finding the real source of the deployments
Investigating `k8s/base/kustomization.yaml` revealed the actual reason: the `resources:` list in that file did **not** include `account-deployment.yaml`, `payment-deployment.yaml`, or any of the other per-service deployment YAML files at all. Those files were leftover artifacts from an earlier stage of the project and were never wired into the build. The real Deployment objects for account-service, payment-service, transaction-service, fraud-detection-service, notification-service, and api-gateway were all being generated dynamically from a shared Helm chart at `charts/microservice`, referenced through a `helmCharts:` block with per-service `valuesInline` overrides. This explained why editing the standalone deployment YAML files had zero effect — those files were never part of the actual build pipeline.

### Step 6 — The correct fix
With the real source identified, three files were changed instead:
1. **`charts/microservice/templates/deployment.yaml`** — added an optional `initContainers` block, only rendered if the Helm values provide one (`{{- if .Values.initContainers }} ... {{- end }}`), so it doesn't affect services that don't need it.
2. **`k8s/base/kustomization.yaml`** — added a `wait-for-mysql` init container definition (via `valuesInline.initContainers`) specifically to the `account-service`, `payment-service`, and `transaction-service` chart entries — the three services that actually depend on MySQL.
3. **`k8s/overlays/dev/kustomization.yaml`** — changed the hard-coded `imagePullPolicy: IfNotPresent` patch to `imagePullPolicy: Always`, so future image rebuilds are picked up automatically on the next `kubectl apply`, without needing a manual rollout restart.

Matching Hikari settings were also added to `payment-service` and `transaction-service`'s `application.yaml` files, mirroring what had already been done for account-service.

### Step 7 — Verification
Before applying anything to the cluster, the rendered manifest was checked directly:
```
kubectl kustomize k8s/overlays/dev --enable-helm --load-restrictor LoadRestrictionsNone > k8s/overlays/dev/banking-rendered.yaml
Select-String "initContainers" k8s/overlays/dev/banking-rendered.yaml
```
This confirmed all three `wait-for-mysql` init container blocks were now present in the actual manifest that would be applied — closing the loop on the earlier problem where edits weren't taking effect.

After applying the manifest and restarting the three deployments, the new pods briefly showed `Init:0/1` status while waiting for MySQL, which had also just restarted and needed about 90 seconds to reinitialize (confirmed directly from MySQL's own logs, which showed it "ready for connections" only after that delay). Once MySQL finished starting, all three init containers completed and the application containers started normally.

### Result
Twelve-plus minutes after the fix was applied, `account-service`, `payment-service`, and `transaction-service` all showed `RESTARTS: 0`, compared to 29, 50, and 50 restarts respectively before the fix. The crash-loop was confirmed resolved.

### What's still open
- `payment-service` and `transaction-service` are running on their pre-fix images; only the Kubernetes-level init container protection is active for them, not yet the Hikari-level protection (blocked on an unresolved `.\mvnw` build error).
- MySQL's deployment has no PersistentVolume, so its data is wiped and it fully reinitializes on every restart — a likely contributor to why MySQL itself was intermittently unavailable in the first place, and a separate issue from the crash-loop that was just fixed.



-----------------------------------------------------------------------------------------------------------


Mình đã có sẵn nội dung 3 file từ tin nhắn trước (charts\microservice\templates\deployment.yaml, k8s\base\kustomization.yaml, k8s\overlays\dev\kustomization.yaml) — để tiết kiệm thời gian, mình sửa luôn 3 file này và gửi lại full nội dung, bạn dán đè. Chỉ có payment-service và transaction-service's application.yaml là mình chưa thấy — 2 file đó bạn gửi lệnh mở + paste như bạn muốn.


notepad charts\microservice\templates\deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.serviceName }}
  labels:
    app: {{ .Values.serviceName }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.serviceName }}
  template:
    metadata:
      labels:
        app: {{ .Values.serviceName }}
    spec:
      # New: optional initContainers block, only rendered if the chart caller
      # (kustomization.yaml valuesInline) sets .Values.initContainers.
      # Purpose: lets account/payment/transaction services wait for MySQL
      # to accept TCP connections on port 3306 before the app container starts,
      # preventing the Hibernate "Connection refused" crash-loop on boot.
      {{- if .Values.initContainers }}
      initContainers:
        {{- toYaml .Values.initContainers | nindent 8 }}
      {{- end }}
      containers:
        - name: {{ .Values.serviceName }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy | default "IfNotPresent" }}
          ports:
            - containerPort: {{ .Values.service.port }}
          envFrom:
            {{- if .Values.configMapName }}
            - configMapRef:
                name: {{ .Values.configMapName }}
            {{- end }}
            {{- if .Values.secretName }}
            - secretRef:
                name: {{ .Values.secretName }}
            {{- end }}
          resources:
            requests:
              cpu: {{ .Values.resources.requests.cpu }}
              memory: {{ .Values.resources.requests.memory }}
            limits:
              cpu: {{ .Values.resources.limits.cpu }}
              memory: {{ .Values.resources.limits.memory }}
			  
			  
			  
notepad k8s\base\kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - account-configmap.yaml
  - account-secret.yaml
  - mysql-deployment.yaml
  - mysql-service.yaml
  - zookeeper-deployment.yaml
  - kafka-deployment.yaml
  - redis-deployment.yaml
helmGlobals:
  chartHome: ../../charts
helmCharts:
  - name: microservice
    releaseName: account-service
    namespace: banking-system
    valuesInline:
      serviceName: account-service
      configMapName: account-config
      secretName: account-secret
      image:
        repository: your-registry/account-service
      service:
        port: 8081
      # New: wait-for-mysql initContainer. Purpose: block app startup until
      # MySQL is actually accepting connections on 3306, fixing the
      # Hibernate "Connection refused" crash-loop (account-service was
      # restarting because Spring context init failed with no retry).
      initContainers:
        - name: wait-for-mysql
          image: busybox:1.36
          command: ['sh', '-c', 'until nc -z -w2 mysql-service 3306; do echo waiting for mysql-service; sleep 3; done']
  - name: microservice
    releaseName: payment-service
    namespace: banking-system
    valuesInline:
      serviceName: payment-service
      configMapName: payment-service-config
      secretName: account-secret
      image:
        repository: your-registry/payment-service
      service:
        port: 8083
      env:
        DB_HOST: mysql-service
        DB_PORT: "3306"
        DB_NAME: payment_db
        KAFKA_HOST: kafka-service
        KAFKA_PORT: "9092"
      # New: same wait-for-mysql fix as account-service (payment-service hits
      # the identical MySQL-not-ready crash-loop).
      initContainers:
        - name: wait-for-mysql
          image: busybox:1.36
          command: ['sh', '-c', 'until nc -z -w2 mysql-service 3306; do echo waiting for mysql-service; sleep 3; done']
  - name: microservice
    releaseName: fraud-detection-service
    namespace: banking-system
    valuesInline:
      serviceName: fraud-detection-service
      configMapName: fraud-detection-service-config
      secretName: account-secret
      image:
        repository: your-registry/fraud-detection-service
      service:
        port: 8084
      env:
        KAFKA_HOST: kafka-service
        KAFKA_PORT: "9092"
        REDIS_HOST: redis-service
        REDIS_PORT: "6379"
  - name: microservice
    releaseName: transaction-service
    namespace: banking-system
    valuesInline:
      serviceName: transaction-service
      configMapName: transaction-service-config
      secretName: account-secret
      image:
        repository: your-registry/transaction-service
      service:
        port: 8082
      env:
        DB_HOST: mysql-service
        DB_PORT: "3306"
        DB_NAME: transaction_db
        KAFKA_HOST: kafka-service
        KAFKA_PORT: "9092"
      # New: same wait-for-mysql fix as account-service (transaction-service
      # hits the identical MySQL-not-ready crash-loop).
      initContainers:
        - name: wait-for-mysql
          image: busybox:1.36
          command: ['sh', '-c', 'until nc -z -w2 mysql-service 3306; do echo waiting for mysql-service; sleep 3; done']
  - name: microservice
    releaseName: notification-service
    namespace: banking-system
    valuesInline:
      serviceName: notification-service
      configMapName: notification-service-config
      secretName: account-secret
      image:
        repository: your-registry/notification-service
      service:
        port: 8085
      env:
        KAFKA_HOST: kafka-service
        KAFKA_PORT: "9092"
  - name: microservice
    releaseName: api-gateway
    namespace: banking-system
    valuesInline:
      serviceName: api-gateway
      configMapName: api-gateway-config
      image:
        repository: your-registry/api-gateway
      service:
        port: 8080
      env:
        REDIS_HOST: redis-service
        REDIS_PORT: "6379"
		
		

notepad k8s\overlays\dev\kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 1
    target:
      kind: Deployment
  - patch: |-
      - op: add
        path: /metadata/namespace
        value: banking-dev
    target:
      kind: Deployment
  - patch: |-
      - op: add
        path: /metadata/namespace
        value: banking-dev
    target:
      kind: Service
  - patch: |-
      - op: add
        path: /metadata/namespace
        value: banking-dev
    target:
      kind: ConfigMap
  - patch: |-
      - op: add
        path: /metadata/namespace
        value: banking-dev
    target:
      kind: Secret
  - patch: |-
      # Old: IfNotPresent caused kubelet to keep using a stale cached image
      # after rebuild/push, so newly built code never actually ran until a
      # manual "kubectl rollout restart".
      # - op: add
      #   path: /spec/template/spec/containers/0/imagePullPolicy
      #   value: IfNotPresent
      # New: Always forces kubelet to re-pull from the local registry every
      # deploy, so `docker push` + `kubectl apply` alone is enough.
      - op: add
        path: /spec/template/spec/containers/0/imagePullPolicy
        value: Always
    target:
      kind: Deployment
  - patch: |-
      - op: add
        path: /spec/template/spec/containers/0/resources
        value:
          limits:
            cpu: "1"
            memory: 1Gi
          requests:
            cpu: 200m
            memory: 256Mi
    target:
      kind: Deployment
images:
  - name: your-registry/account-service
    newName: localhost:5000/account-service
    newTag: dev-latest
  - name: your-registry/payment-service
    newName: localhost:5000/payment-service
    newTag: dev-latest
  - name: your-registry/fraud-detection-service
    newName: localhost:5000/fraud-detection-service
    newTag: dev-latest
  - name: your-registry/transaction-service
    newName: localhost:5000/transaction-service
    newTag: dev-latest
  - name: your-registry/notification-service
    newName: localhost:5000/notification-service
    newTag: dev-latest
  - name: your-registry/api-gateway
    newName: localhost:5000/api-gateway
    newTag: dev-latest
	
	
	
	
2 file còn lại — mở và gửi mình toàn bộ nội dung:
powershell
notepad payment-service\src\main\resources\application.yaml
notepad transaction-service\src\main\resources\application.yaml


notepad payment-service\src\main\resources\application.yaml


spring:
  application:
    name: payment-service
  datasource:
    # Old config (local)
    # url: jdbc:mysql://localhost:3306/payment_db?createDatabaseIfNotExist=true
    # username: root
    # password: root
    # driver-class-name: com.mysql.cj.jdbc.Driver
    # New config (Kubernetes environment variables)
    url: jdbc:mysql://${DB_HOST}:${DB_PORT}/${DB_NAME}?createDatabaseIfNotExist=true
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
    # New: Hikari tuning. Purpose: initialization-fail-timeout=-1 stops the
    # pool from throwing immediately if MySQL isn't reachable yet at startup,
    # letting Spring context init finish and Hikari retry connections in the
    # background instead of crashing the app (root cause of the
    # payment-service crash-loop, same as account-service).
    hikari:
      connection-timeout: 30000
      initialization-fail-timeout: -1
      max-lifetime: 600000
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect
  kafka:
    # Old config (local)
    # bootstrap-servers: localhost:9092
    # New config (Kubernetes environment variables)
    bootstrap-servers: ${KAFKA_HOST}:${KAFKA_PORT}
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
server:
  port: 8083
# Razorpay Configuration
razorpay:
  key-id: rzp_xxxxxx
  key-secret:
  webhook-secret:
management:
  endpoints:
    web:
      exposure:
        include: health,info
		
		

notepad transaction-service\src\main\resources\application.yaml


server:
  port: 8082
spring:
  application:
    name: transaction-service
  datasource:
    # Old config (local)
    #url: jdbc:mysql://localhost:3306/transaction_db?createDatabaseIfNotExist=true
    #username: root
    #password: root
    #driver-class-name: com.mysql.cj.jdbc.Driver
    # New config (Kubernetes environment variables)
    url: jdbc:mysql://${DB_HOST}:${DB_PORT}/${DB_NAME}?createDatabaseIfNotExist=true
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
    # New: Hikari tuning. Purpose: initialization-fail-timeout=-1 stops the
    # pool from throwing immediately if MySQL isn't reachable yet at startup,
    # letting Spring context init finish and Hikari retry connections in the
    # background instead of crashing the app (root cause of the
    # transaction-service crash-loop, same as account-service).
    hikari:
      connection-timeout: 30000
      initialization-fail-timeout: -1
      max-lifetime: 600000
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect
  kafka:
    # Old config (local)
    #bootstrap-servers: localhost:9092
    # New config (Kubernetes environment variables)
    bootstrap-servers: ${KAFKA_HOST}:${KAFKA_PORT}
    consumer:
      group-id: transaction-service-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring:
          json:
            trusted:
              packages: "*"
            use:
              type:
                headers: false
            value:
              default:
                type: java.util.HashMap
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
management:
  endpoints:
    web:
      exposure:
        include: health, info
account:
  service:
    # Old config (local)
    # url: http://localhost:8081
    #url: http://localhost:8081
    # New config (Kubernetes environment)
    url: http://account-service:8081
	
	

Cả 5 file đã sửa xong. Sau khi bạn save tất cả, chạy build + verify + deploy:


cd account-service
.\mvnw clean package -DskipTests
docker build -t localhost:5000/account-service:dev-latest .
docker push localhost:5000/account-service:dev-latest

cd ..\payment-service
.\mvnw clean package -DskipTests
docker build -t localhost:5000/payment-service:dev-latest .
docker push localhost:5000/payment-service:dev-latest

cd ..\transaction-service
.\mvnw clean package -DskipTests
docker build -t localhost:5000/transaction-service:dev-latest .
docker push localhost:5000/transaction-service:dev-latest

cd ..
kubectl kustomize k8s\overlays\dev --enable-helm --load-restrictor LoadRestrictionsNone > k8s\overlays\dev\banking-rendered.yaml


Bắt buộc verify trước khi apply (để không lặp lại vòng lặp đã tốn thời gian trước đó):


Select-String "initContainers" k8s\overlays\dev\banking-rendered.yaml -Context 0,5


Phải thấy 3 khối wait-for-mysql (account, payment, transaction). Nếu đủ 3 khối, apply:


kubectl apply -f k8s\overlays\dev\banking-rendered.yaml
kubectl rollout restart deployment/account-service deployment/payment-service deployment/transaction-service -n banking-dev


----

Lỗi này nghiêm trọng hơn lỗi UI — daemon Docker Desktop Linux Engine đang trả về 500 Internal Server Error cho chính API core (/images/json), nghĩa là backend WSL2/Linux VM của Docker Desktop đang gặp sự cố thật sự, không chỉ là giao diện bị treo.

Kiểm tra thêm 1 lệnh trước khi quyết định restart, để biết phạm vi ảnh hưởng:

powershell
docker version
kubectl get pods -n banking-dev
Nếu docker version cũng lỗi tương tự, và kubectl get pods lỗi luôn (không kết nối được) → toàn bộ Docker Desktop backend đã treo, bắt buộc phải restart Docker Desktop.
Nếu kubectl get pods vẫn chạy được bình thường → cluster Kubernetes có thể vẫn đang chạy ổn, chỉ có phần Docker Engine API (dùng cho docker build, docker images...) bị lỗi riêng — ít nghiêm trọng hơn.

Chạy 2 lệnh trên và cho mình biết kết quả. Nếu buộc phải restart Docker Desktop, nhắc lại điều quan trọng đã nói trước đó: vì MySQL chưa có PersistentVolume, restart sẽ làm mất toàn bộ dữ liệu MySQL hiện tại (database sẽ về trạng thái rỗng, createDatabaseIfNotExist=true sẽ tự tạo lại schema rỗng) — nhưng sẽ không ảnh hưởng gì tới fix crash-loop vừa làm (initContainer + Helm chart đã lưu trong file, không mất khi restart Docker Desktop).




--------------------------------------------------

Dựa trên toàn bộ file bạn đã gửi (Kafka/Zookeeper event-driven giữa account/transaction/fraud-detection, Razorpay ở payment-service, Redis chỉ ở transaction-service, DB riêng từng service, Helm chart vừa hoàn thiện), đây là các tính năng đáng đầu tư — xếp theo mức độ "impact cho hệ thống banking" và mức độ dễ làm:

1. Idempotency cho Payment (ưu tiên cao nhất — bug tiền thật)
Hiện PaymentController nhận request tạo payment nhưng chưa thấy cơ chế chặn duplicate (network retry, user bấm 2 lần, Razorpay webhook gọi lại). Thêm 1 cột idempotency_key unique + check trước khi tạo.

payment-service/src/main/java/com/banking/paymentservice/controller/PaymentController.java
payment-service/src/main/java/com/banking/paymentservice/entity/Payment.java
payment-service/src/main/java/com/banking/paymentservice/service/PaymentService.java

2. Circuit breaker + retry (Resilience4j) cho lời gọi liên service
fraud-detection-service và transaction-service đều gọi thẳng AccountServiceClient — nếu account-service chậm/down, cả chuỗi sập theo. Thêm Resilience4j (circuit breaker, retry, timeout, fallback).

fraud-detection-service/src/main/java/com/banking/frauddetectionservice/client/AccountServiceClient.java
transaction-service/src/main/java/com/banking/transactionservice/client/AccountServiceClient.java
pom.xml của cả 2 service (thêm dependency)
application.yaml của cả 2 service (cấu hình resilience4j.circuitbreaker)

3. Dead Letter Topic cho Kafka consumer
AccountEventConsumer, FraudDetectionEventConsumer, TransactionEventConsumer hiện chưa rõ xử lý gì khi message lỗi (deserialize fail, exception) — dễ mất event hoặc consumer bị stuck retry vô hạn. Thêm error handler đẩy sang DLT.

account-service/src/main/java/com/banking/accountservice/service/AccountEventConsumer.java
fraud-detection-service/src/main/java/com/banking/frauddetectionservice/service/FraudDetectionEventConsumer.java
transaction-service/src/main/java/com/banking/transactionservice/service/TransactionEventConsumer.java
application.yaml mỗi service (thêm spring.kafka.consumer.error-handler / DefaultErrorHandler bean)

4. Database migration có version (Flyway)
Đang thấy hibernate.ddl-auto/show-sql: true trong application.yaml (screenshot bạn gửi) — nghĩa là Hibernate tự sinh/sửa schema, rất rủi ro khi lên prod (mất data, đổi schema không kiểm soát). Chuyển sang Flyway.

account-service/src/main/resources/application.yaml (+ tương tự payment/transaction-service)
Thêm mới: account-service/src/main/resources/db/migration/V1__init.sql (và tương tự payment/transaction)
pom.xml mỗi service có DB (thêm flyway-core, flyway-mysql)

5. Observability: Prometheus + Grafana + tracing
Hiện không thấy metrics/tracing nào — với hệ thống nhiều service gọi chéo nhau qua Kafka + REST, không có tracing thì debug production cực khó.

pom.xml mỗi service (thêm micrometer-registry-prometheus, micrometer-tracing-bridge-otel)
application.yaml mỗi service (management.endpoints.web.exposure.include: prometheus,health)
banking-system-helm/charts/*/templates/deployment.yaml (thêm annotation prometheus.io/scrape)
Mới: banking-system-helm/templates/monitoring/ nếu dùng kube-prometheus-stack (ServiceMonitor)

6. Secrets không để plaintext root/root nữa
account-secret.yaml đang để DB_USERNAME/DB_PASSWORD: root dạng stringData commit thẳng vào git — rủi ro bảo mật thật sự cho banking system.

k8s/base/account-secret.yaml (và các secret tương tự nếu có)
banking-system-helm/charts/*/templates/secret.yaml
banking-system-helm/values-prod.yaml (đã có chỗ chờ sẵn values-secrets.yaml không commit — nên dùng đúng chỗ này, hoặc nâng cấp lên Sealed Secrets/External Secrets Operator)

7. Bật lại Persistence cho MySQL + backup
Hiện mysql-deployment.yaml không có volume — pod restart là mất sạch dữ liệu banking. Đã có sẵn field mysql.persistence.enabled trong chart (đang tắt).

banking-system-helm/values.yaml / values-prod.yaml (bật persistence.enabled: true)
banking-system-helm/templates/infra-mysql.yaml (đã hỗ trợ PVC sẵn, chỉ cần bật)
Cân nhắc thêm mới: banking-system-helm/templates/mysql-backup-cronjob.yaml

8. CI/CD tự động build → push → helm upgrade
Hiện build/push image vào localhost:5000 có vẻ đang làm tay.

Mới: .github/workflows/ci-cd.yaml (hoặc Jenkinsfile nếu bạn dùng Jenkins)
banking-system-helm/values-dev.yaml / values-prod.yaml (image tag được CI set qua --set)

Bạn muốn mình bắt đầu implement cái nào trước? Với hệ thống banking, mình sẽ ưu tiên thứ tự: #1 idempotency payment → #6 secrets → #4 Flyway → #2 circuit breaker → #3 DLQ Kafka → #7 persistence → #5 observability → #8 CI/CD.


--------------------------------------------------------------------------------------------------------------------------------------


Nhìn qua cấu trúc project eazybytes này (đây là template banking microservices rất nổi tiếng của khóa "Master Microservices with Spring Boot"), nó phát triển tuần tự qua từng "section" — đúng là một roadmap kỹ thuật rất hay để áp dụng vào banking-system của bạn. So với project hiện tại của bạn (account/payment/transaction/fraud-detection/notification/api-gateway, đã có Kafka + K8s/Helm/Kustomize khá xịn rồi), đây là các kỹ thuật đáng lấy, xếp theo độ ưu tiên:

1. Config Server (section6 v2) — nên làm sớm
Tách hết application.yml của từng service ra một repo/folder config riêng, service chỉ giữ spring.config.import=configserver:. Giúp đổi config (DB, Kafka, Razorpay keys...) mà không cần rebuild image — rất hợp với việc bạn đang có nhiều issue liên quan tới config sai giữa các overlay (dev/qa/prod).

2. Eureka / Service Discovery (section7)
Nếu bạn đang gọi service qua tên K8s Service (DNS) thì có thể bỏ qua bước này — K8s tự làm discovery rồi. Chỉ cần nếu bạn muốn load-balancing phía client hoặc chạy ngoài K8s (local dev).

3. Feign Client + Fallback/Circuit Breaker (section8 → section_10)
Đây là cái đáng áp dụng nhất cho bạn: AccountsController gọi sang Cards/Loans qua CardsFeignClient/LoansFeignClient, có CardsFallback/LoansFallback (Resilience4j). Bạn có thể áp cho api-gateway hoặc transaction-service khi gọi account-service/fraud-detection-service — tránh cascading failure khi 1 service down.

4. Gateway + Security bằng Keycloak (section9 → section_12)
GatewayserverApplication + SecurityConfig/KeycloakRoleConverter — OAuth2 resource server, route-based role authorization ngay ở gateway. Rất hợp vì bạn đã có api-gateway với RateLimiterConfig sẵn, thêm security layer vào đây là tự nhiên.

5. Observability stack (section_11)
Grafana + Loki (log) + Tempo (tracing) + Prometheus + Alloy — đóng gói sẵn dạng Helm chart. Vì bạn đã có K8s + Helm/Kustomize rồi, phần này copy gần như nguyên chart về là chạy được, giúp debug các bug crash-loop kiểu bạn từng gặp (MySQL connection refused) nhanh hơn nhiều nhờ có trace + log tập trung.

6. Event-driven qua Spring Cloud Function (section_13/14, message service)
Thay vì mỗi service tự viết @KafkaListener, họ dùng Spring Cloud Function (AccountsFunctions, MessageFunctions) để định nghĩa message xử lý như function thuần, cấu hình binding qua YAML. Cân nhắc nếu muốn giảm boilerplate Kafka listener trong notification-service/fraud-detection-service.

7. BOM dùng chung (section_20, eazy-bom)
Một module Maven BOM quản lý version chung (Spring Boot, dependencies) cho tất cả service — tránh version-drift giữa 6 module của bạn, và fix nhanh hơn các lỗi pom.xml sai artifact-id mà bạn từng gặp.

Bạn muốn mình đi sâu vào cái nào trước — Feign + fallback, hay observability stack (vì nó liên quan trực tiếp tới mấy vụ crash-loop MySQL bạn đã fix gần đây)?



--------------------------------------------------------------------------------------------------------------------------------------------------------------


Vì bạn đã có nền khá tốt rồi (Helm chart chung + Kustomize overlays, đã fix xong crash-loop), mình sẽ xếp theo tỷ lệ "công sức thấp / giá trị cao" trước:

Nhóm 1 — làm ngay, nhanh (vài giờ), giá trị cao:

PersistentVolume cho MySQL — bạn đã tự flag việc này rồi (mysql-deployment.yaml chưa có PV nên data mất mỗi lần restart, nghi là nguồn gây flaky trước đó). Đây là việc còn treo, nên dọn trước khi làm gì khác, vì nếu không mọi lần restart pod vẫn phải chờ MySQL init lại ~90s.
Liveness/Readiness probes cho account/payment/transaction/fraud-detection/notification — hiện Helm chart của bạn (charts/microservice/templates/deployment.yaml) có thể chưa có. Thêm readinessProbe (/actuator/health/readiness) sẽ giúp K8s không route traffic vào pod chưa sẵn sàng — trực tiếp giải quyết đúng loại lỗi bạn từng gặp (pod "Running" nhưng thực chất chưa kết nối được DB).
Resource requests/limits — bạn từng nghi MySQL bị OOMKilled (exit code 137). Đặt resources.requests/limits rõ ràng cho MySQL + các service sẽ giúp scheduler đặt pod hợp lý hơn và tránh OOMKill âm thầm.
K8s Secret thay vì plaintext cho DB password, Razorpay keys — nếu hiện đang để trong values.yaml/ConfigMap dạng plaintext thì đây là việc nhanh và quan trọng về bảo mật.

Nhóm 2 — làm sau, công sức vừa, giá trị cao:

PodDisruptionBudget (PDB) cho mỗi service — vài dòng YAML, đảm bảo khi node bị drain/update thì không mất hết pod cùng lúc.
HPA (Horizontal Pod Autoscaler) — dễ thêm vào Helm chart chung vì bạn đã có 1 chart dùng cho tất cả service (charts/microservice), chỉ cần bật autoscaling.enabled trong values từng service.

Nhóm 3 — lớn hơn, để sau:

Observability (Prometheus/Grafana/Loki/Tempo) — giá trị rất cao để debug nhanh các bug kiểu MySQL connection refused bạn từng gặp, nhưng setup tốn thời gian hơn (nhiều CRD, ServiceMonitor...).
NetworkPolicy, Ingress, GitOps — để cuối vì ít cấp bách hơn với quy mô dev hiện tại.


--------------------------------------------------------------------------------------------------------------------------------------------------------------


kubectl kustomize k8s\overlays\dev --enable-helm --load-restrictor LoadRestrictionsNone > k8s\overlays\dev\banking-rendered.yaml
kubectl apply -f k8s\overlays\dev\banking-rendered.yaml
kubectl get pvc -n banking-system