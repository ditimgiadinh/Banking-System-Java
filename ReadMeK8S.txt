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