---
title: apiserver 中的webhook开发教程
author: admin
type: post
date: 2023-08-03T12:48:58+00:00
url: /archives/34883
categories:
 - 程序开发
tags:
 - k8s
 - webhook

---
k8s: v1.27.3

## 什么是准入控制插件？[][1] {#what-are-they.wp-block-heading}

**准入控制器** 是一段代码，它会在请求通过**认证**和**鉴权**之后、对象被持久化之前拦截到达 API 服务器的请求。![](https://blogstatic.haohtml.com/uploads/2023/08/6ca5dd6b207691069de1cf4df59cc6ad.png)

准入控制器可以执行 **变更（Mutating）** 和或 **验证（Validating）** 操作。 变更（mutating）控制器可以根据被其接受的请求更改相关对象；验证（validating）控制器则不行。

准入控制器限制创建、删除、修改对象的请求。 准入控制器也可以阻止自定义动作，例如通过 API 服务器代理连接到 Pod 的请求。 准入控制器**不会** （也不能）阻止读取（**get**、**watch** 或 **list**）对象的请求。

某些控制器既是变更准入控制器又是验证准入控制器。如果两个阶段之一的任何一个控制器拒绝了某请求，则整个请求将立即被拒绝，并向最终用户返回错误。

Kubernetes 1.27 中的准入控制器由下面的[列表][2]组成， 并编译进 `kube-apiserver` 可执行文件，并且只能由集群管理员配置。 在该列表中，有两个特殊的控制器：`MutatingAdmissionWebhook` 和 `ValidatingAdmissionWebhook`。 它们根据 API 中的配置， 分别执行变更和验证[准入控制 webhook][3]。

参考： [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/)

> 同类型的webhook的顺序无关紧要。

# 什么是准入 Webhook？ 

除了[内置的 admission 插件][4]， 准入插件可以作为扩展独立开发，并以运行时所配置的 Webhook 的形式运行。 此页面描述了如何构建、配置、使用和监视准入 Webhook。

准入 Webhook 是一种用于接收准入请求并对其进行处理的 HTTP 回调机制。 可以定义两种类型的准入 webhook，即 [修改性质的准入 Webhook(Mutating admission)][5] 和 [验证性质的准入 Webhook ( Validating admission)][6] 。![apiserver-webhook](https://blogstatic.haohtml.com/uploads/2023/08/9b1359b6c5569ce45ec9da2ad8423604.png)

**Mutalting Webhook** 会先被调用，它们可以更改发送到 API 服务器的对象以执行自定义的设置默认值操作。

在完成了所有对象修改并且 API 服务器也验证了所传入的对象之后，紧接着 **Validating Webhook** 会被调用，并通过拒绝请求的方式来强制实施自定义的策略。

一定要搞明白这两类webhook的执行先后顺序,它们均位于`Authentication` 和 `Authorization` 之后。

# webhook配置 

Webhook 可以在配置中的 `admissionReviewVersions` 字段指定可接受的 `AdmissionReview` 对象版本：

```
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
  clientConfig:
    service:
      namespace: "example-namespace"
      name: "example-service"
    caBundle: <CA_BUNDLE>
  admissionReviewVersions: ["v1"]
  sideEffects: None
  timeoutSeconds: 5
```

创建 Webhook 配置时，`admissionReviewVersions` 是必填字段。 Webhook 必须支持至少一个当前和以前的 API 服务器都可以解析的 `AdmissionReview` 版本。

API 服务器将发送的是 `admissionReviewVersions` 列表中所支持的第一个 `AdmissionReview` 版本。如果 API 服务器不支持列表中的任何版本，则不允许创建配置。

如果 API 服务器遇到以前创建的 Webhook 配置，并且不支持该 API 服务器知道如何发送的任何 `AdmissionReview` 版本，则调用 Webhook 的尝试将失败，并依据[失败策略][7]进行处理。

要注册准入 Webhook，请创建 `MutatingWebhookConfiguration` 或 `ValidatingWebhookConfiguration` API 对象。 `MutatingWebhookConfiguration` 或`ValidatingWebhookConfiguration` 对象的名称必须是有效的 [DNS 子域名][8]。

每种配置可以包含一个或多个 Webhook。如果在单个配置中指定了多个 Webhook，则应为每个 Webhook 赋予一个唯一的名称。 这是必需的，以使生成的审计日志和指标更易于与激活的配置相匹配。

## **匹配请求-规则** 

每个 Webhook 必须指定用于确定是否应将对 apiserver 的请求发送到 webhook 的规则列表。 每个规则都指定一个或多个 operations、apiGroups、apiVersions 和 resources 以及资源的 scope：

 * `operations` 列出一个或多个要匹配的操作。 可以是 `CREATE`、`UPDATE`、`DELETE`、`CONNECT` 或 `*` 以匹配所有内容。
 * `apiGroups` 列出了一个或多个要匹配的 API 组。`""` 是核心 API 组。`"*"` 匹配所有 API 组。
 * `apiVersions` 列出了一个或多个要匹配的 API 版本。`"*"` 匹配所有 API 版本。
 * `resources` 列出了一个或多个要匹配的资源。
 * `"*"` 匹配所有资源，但不包括子资源。
 * `"*/*"` 匹配所有资源，包括子资源。
 * `"pods/*"` 匹配 pod 的所有子资源。
 * `"*/status"` 匹配所有 status 子资源。
 * `scope` 指定要匹配的范围。有效值为 `"Cluster"`、`"Namespaced"` 和 `"*"`。 子资源匹配其父资源的范围。默认值为 `"*"`。
 * `"Cluster"` 表示只有集群作用域的资源才能匹配此规则（API 对象 Namespace 是集群作用域的）。
 * `"Namespaced"` 意味着仅具有名字空间的资源才符合此规则。
 * `"*"` 表示没有作用域限制。

如果传入请求与任何 Webhook `rules` 的指定 `operations`、`groups`、`versions`、 `resources` 和 `scope` 匹配，则该请求将发送到 Webhook。

## 请求认证 

API 服务器确定请求应发送到 Webhook 后，它需要知道如何调用 webhook。 此信息在 Webhook 配置的 `clientConfig` 节中指定。

Webhook 可以通过 `URL` 或 `服务引用` 两种方式调用，并且可以选择包含自定义 CA 包，以用于验证 TLS 连接。

### URL 

`url` 以标准 URL 形式给出 Webhook 的位置（`scheme://host:port/path`）。

`host` 不应引用集群中运行的服务；通过指定 `service` 字段来使用服务引用。 主机可以通过某些 API 服务器中的外部 DNS 进行解析。 （例如，`kube-apiserver` 无法解析集群内 DNS，因为这将违反分层规则）。`host` 也可以是 IP 地址。

请注意，将 `localhost` 或 `127.0.0.1` 用作 `host` 是有风险的， 除非你非常小心地在所有运行 apiserver 的、可能需要对此 Webhook 进行调用的主机上运行。这样的安装方式可能不具有可移植性，即很难在新集群中启用。

**scheme 必须为 “https”；URL 必须以 “https://” 开头。**

> 使用用户或基本身份验证（例如：”user:password@”）是不允许的。 使用片段（”#…”）和查询参数（”?…”）也是不允许的。

这是配置为调用 URL 的修改性质的 Webhook 的示例 （**并且期望使用系统信任根证书来验证 TLS 证书，因此不指定 caBundle**）：

```
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  clientConfig:
    url: "https://my-webhook.example.com:9443/my-webhook-path"
```

参考： [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#contacting-the-webhook](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#contacting-the-webhook)

### 服务引用 

`clientConfig` 内部的 Service 是对该 Webhook 服务的引用。 如果 Webhook 在集群中运行，则应使用 `service` 而不是 `url`。 服务的 `namespace` 和 `name` 是必需的。 `port` 是可选的，默认值为 443。`path` 是可选的，默认为 “/”。

这是一个 `mutating Webhook` 的示例，该 mutating Webhook 配置为在子路径 “/my-path” 端口 “1234” 上调用服务，并使用自定义 CA 包针对 ServerName `my-service-name.my-service-namespace.svc` 验证 TLS 连接：

```
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  clientConfig:
    caBundle: <CA_BUNDLE>
    service:
      namespace: my-service-namespace
      name: my-service-name
      path: /my-path
      port: 1234
```

> **说明：**
>
> 你必须在以上示例中将 `<CA_BUNDLE>` 替换为一个有效的 VA 证书包， 这是一个用 PEM 编码的 CA 证书包，用于校验 Webhook 的服务器证书。

参考：

# 请求与响应 

## 请求 

当注册一个 webhook 成功后，APIServer 中的 webhook 发送 POST 请求时，请设置 `Content-Type: application/json` 并对 `admission.k8s.io` API 组中的 `AdmissionReview` 对象进行序列化，将所得到的 JSON 作为请求的主体。

```
apiVersion: admission.k8s.io/v1
kind: AdmissionReview
request:
  # 唯一标识此准入回调的随机 uid
  uid: 705ab4f5-6393-11e8-b7cc-42010a800002

  # 传入完全正确的 group/version/kind 对象
  kind:
    group: autoscaling
    version: v1
    kind: Scale

  # 修改 resource 的完全正确的的 group/version/kind
  resource:
    group: apps
    version: v1
    resource: deployments

  # subResource（如果请求是针对 subResource 的）
  subResource: scale

  # 在对 API 服务器的原始请求中，传入对象的标准 group/version/kind
  # 仅当 Webhook 指定 `matchPolicy: Equivalent` 且将对 API 服务器的原始请求
  # 转换为 Webhook 注册的版本时，这才与 `kind` 不同。
  requestKind:
    group: autoscaling
    version: v1
    kind: Scale

  # 在对 API 服务器的原始请求中正在修改的资源的标准 group/version/kind
  # 仅当 Webhook 指定了 `matchPolicy：Equivalent` 并且将对 API 服务器的原始请求转换为
  # Webhook 注册的版本时，这才与 `resource` 不同。
  requestResource:
    group: apps
    version: v1
    resource: deployments

  # subResource（如果请求是针对 subResource 的）
  # 仅当 Webhook 指定了 `matchPolicy：Equivalent` 并且将对
  # API 服务器的原始请求转换为该 Webhook 注册的版本时，这才与 `subResource` 不同。
  requestSubResource: scale

  # 被修改资源的名称
  name: my-deployment

  # 如果资源是属于名字空间（或者是名字空间对象），则这是被修改的资源的名字空间
  namespace: my-namespace

  # 操作可以是 CREATE、UPDATE、DELETE 或 CONNECT
  operation: UPDATE

  userInfo:
    # 向 API 服务器发出请求的经过身份验证的用户的用户名
    username: admin

    # 向 API 服务器发出请求的经过身份验证的用户的 UID
    uid: 014fbff9a07c

    # 向 API 服务器发出请求的经过身份验证的用户的组成员身份
    groups:
      - system:authenticated
      - my-admin-group
    # 向 API 服务器发出请求的用户相关的任意附加信息
    # 该字段由 API 服务器身份验证层填充，并且如果 webhook 执行了任何
    # SubjectAccessReview 检查，则应将其包括在内。
    extra:
      some-key:
        - some-value1
        - some-value2

  # object 是被接纳的新对象。
  # 对于 DELETE 操作，它为 null。
  object:
    apiVersion: autoscaling/v1
    kind: Scale

  # oldObject 是现有对象。
  # 对于 CREATE 和 CONNECT 操作，它为 null。
  oldObject:
    apiVersion: autoscaling/v1
    kind: Scale

  # options 包含要接受的操作的选项，例如 meta.k8s.io/v CreateOptions、UpdateOptions 或 DeleteOptions。
  # 对于 CONNECT 操作，它为 null。
  options:
    apiVersion: meta.k8s.io/v1
    kind: UpdateOptions

  # dryRun 表示 API 请求正在以 `dryrun` 模式运行，并且将不会保留。
  # 带有副作用的 Webhook 应该避免在 dryRun 为 true 时激活这些副作用。
  # 有关更多详细信息，请参见 http://k8s.io/docs/reference/using-api/api-concepts/#make-a-dry-run-request
  dryRun: False
```

参考：

## 响应 

Webhook 使用 HTTP 200 状态码、`Content-Type: application/json` 和一个包含 `AdmissionReview` 对象的 JSON 序列化格式来发送响应。该 `AdmissionReview` 对象与发送的版本相同，且其中包含的 `response` 字段已被有效填充。

`response` 至少必须包含以下字段：

 * `uid`，从发送到 Webhook 的 `request.uid` 中复制而来
 * `allowed`，设置为 `true` 或 `false`

Webhook 允许请求的最简单响应示例：

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true
  }
}
```

Webhook 禁止请求的最简单响应示例：

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false
  }
}
```

当拒绝请求时，Webhook 可以使用 `status` 字段自定义 http 响应码和返回给用户的消息。 有关状态类型的详细信息，请参见 [API 文档][9]。 禁止请求的响应示例，它定制了向用户显示的 HTTP 状态码和消息：

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false,
    "status": {
      "code": 403,
      "message": "You cannot do this because it is Tuesday and your name starts with A"
    }
  }
}
```

当允许请求时，mutating准入 Webhook 也可以选择修改传入的对象。 这是通过在响应中使用 `patch` 和 `patchType` 字段来完成的。 当前唯一支持的 `patchType` 是 `JSONPatch`。 有关更多详细信息，请参见 [JSON patch][10]。 对于 `patchType: JSONPatch`，`patch` 字段包含一个以 base64 编码的 JSON patch 操作数组。

例如，设置 `spec.replicas` 的单个补丁操作将是 `[{"op": "add", "path": "/spec/replicas", "value": 3}]`。

如果以 Base64 形式编码，结果将是 `W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0=`

因此，添加该标签的 Webhook 响应为：

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "patchType": "JSONPatch",
    "patch": "W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0="
  }
}
```

准入 Webhook 可以选择性地返回在 HTTP `Warning` 头中返回给请求客户端的警告消息，警告代码为 299。 警告可以与允许或拒绝的准入响应一起发送。

如果你正在实现返回一条警告的 webhook，则：

 * 不要在消息中包括 “Warning:” 前缀
 * 使用警告消息描述该客户端进行 API 请求时会遇到或应意识到的问题
 * 如果可能，将警告限制为 120 个字符

参考：

# 失败策略 

`failurePolicy` 定义了如何处理准入 Webhook 中无法识别的错误和超时错误。允许的值为 `Ignore` 或 `Fail`。

 * `Ignore` 表示调用 Webhook 的错误将被忽略并且允许 API 请求继续。
 * `Fail` 表示调用 Webhook 的错误导致准入失败并且 API 请求被拒绝。

这是一个修改性质的 webhook，配置为在调用准入 Webhook 遇到错误时拒绝 API 请求：

```
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  failurePolicy: Fail
```

准入 Webhook 所用的默认 `failurePolicy` 是 `Fail`。

参考：

# 开发实战 

上面我们分别介绍了如何注册一个webhook 以及对一个webhook 如何发磅请求与响应。下面我们编写一个例子来测试一下webhook的功能。

参考官方示例：

## 客户端 

生成请求webhook需要的证书

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=YOUR_CA_NAME" -days 3650 -out ca.crt
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=YOUR_SERVER_DNS" -out server.csr
echo "subjectAltName = IP:127.0.0.1" > extfile.cnf
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -extfile extfile.cnf
```

## 注册webhook 

这里以 `MutatingAdmissionWebhok` 为例，创建 `test-mutating-webhook.yaml` 文件

```
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: test-mutating-webhook
webhooks:
  - name: test-mutating-webhook.example.com
    clientConfig:
      caBundle: <CA_CERTIFICATE>
      url: https://127.0.0.1:8080/mutate
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE"]
        resources: ["pods"]
    failurePolicy: Fail
    sideEffects: None
    admissionReviewVersions: ["v1", "v1beta1"]
```

这里的 `webhook` 服务端地址为 `https://127.0.0.1:8080/mutate`（注意是https），并指定了失败策略为`Fail`，修改的资源资源为 `pod`, `sideEffects` 表示请求 `dryRun` 没有副作用。

其中 `` 是根证书的内容，需要执行 `cat ca.crt | base64` 后将输出内容替换到其中。

将webhook在集群中注册

```
$ kubectl apply -f test-mutating-webhook.yaml
mutatingwebhookconfiguration.admissionregistration.k8s.io/test-mutating-webhook created
```

## 服务端 

```
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"

    admissionv1 "k8s.io/api/admission/v1"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    "k8s.io/apimachinery/pkg/runtime/serializer"
)

func main() {
    // 注册 /mutate 路由
    http.HandleFunc("/mutate", mutate)
    // 启动 webhook 服务，证书在上面已经生成过了，直接拿过来用即可
    panic(http.ListenAndServeTLS(":8080", "server.crt", "server.key", nil))
}

// patch
type patchOperation struct {
    Op    string      `json:"op"`
    Path  string      `json:"path"`
    Value interface{} `json:"value,omitempty"`
}

func mutate(w http.ResponseWriter, r *http.Request) {
    fmt.Println("收到 kube-apiserver 发送的请求")

    // 解析 body 为 Pod 对象
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    deserializer := serializer.NewCodecFactory(runtime.NewScheme()).UniversalDeserializer()
    ar := admissionv1.AdmissionReview{}
    if _, _, err := deserializer.Decode(body, nil, &ar); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    var pod corev1.Pod
    if err := json.Unmarshal(ar.Request.Object.Raw, &pod); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    // 添加一个注释
    if pod.ObjectMeta.Annotations == nil {
        pod.ObjectMeta.Annotations = make(map[string]string)
    }
    pod.ObjectMeta.Annotations["mutate"] = "云原生k8s-apiserver-webhook"

    // 构造 Patch 对象
    patch := []patchOperation{
        {
            Op:    "add",
            Path:  "/metadata/annotations",
            Value: pod.ObjectMeta.Annotations,
        },
    }
    patchBytes, err := json.Marshal(patch)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    // 将篡改后的 Patch 对象内容写入 response
    admissionReview := admissionv1.AdmissionReview{
        TypeMeta: metav1.TypeMeta{
            APIVersion: "admission.k8s.io/v1",
            Kind:       "AdmissionReview",
        },
        Response: &admissionv1.AdmissionResponse{
            UID:     ar.Request.UID,
            Allowed: true,
            Patch:   patchBytes,
            PatchType: func() *admissionv1.PatchType {
                pt := admissionv1.PatchTypeJSONPatch
                return &pt
            }(),
        },
    }
    resp, err := json.Marshal(admissionReview)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    if _, err := w.Write(resp); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    fmt.Printf("%#vn", pod.ObjectMeta)
    fmt.Printf("[%s/%s]资源变更成功nn", pod.ObjectMeta.Namespace, pod.ObjectMeta.Name)
}
```

> 这里没有对版本号做检查，生产中需要这一步操作。

此时启动 webserver 服务。

```
$ go run main.go
```

## 测试 

创建一个pod 文件 pod.yaml，并检查是否被修改

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.23
    ports:
    - containerPort: 80
```

查看 pod 信息

```
$ kubectl apply -f pod.yaml
pod/nginx created

$ kubectl get pod nginx -o yaml | grep mutate
    mutate: 云原生k8s-apiserver-webhook
```

再看下webhook 服务端日志

```
$ go run main.go
收到 kube-apiserver 发送的请求
v1.ObjectMeta{Name:"nginx", GenerateName:"", Namespace:"default", SelfLink:"", UID:"", ResourceVersion:"", Generation:0, CreationTimestamp:time.Date(1, time.January, 1, 0, 0, 0, 0, time.UTC), DeletionTimestamp:<nil>, DeletionGracePeriodSeconds:(*int64)(nil), Labels:map[string]string(nil), Annotations:map[string]string{"kubectl.kubernetes.io/last-applied-configuration":"{"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"containers":[{"image":"nginx:1.23","name":"nginx","ports":[{"containerPort":80}]}]}}n", "mutate":"云原生k8s-apiserver-webhook"}, OwnerReferences:[]v1.OwnerReference(nil), Finalizers:[]string(nil), ManagedFields:[]v1.ManagedFieldsEntry{v1.ManagedFieldsEntry{Manager:"kubectl-client-side-apply", Operation:"Update", APIVersion:"v1", Time:time.Date(2023, time.August, 3, 20, 55, 27, 0, time.Local), FieldsType:"FieldsV1", FieldsV1:(*v1.FieldsV1)(0xc00021e510), Subresource:""}}}
[default/nginx]资源变更成功
```

可以看到webhook请求成功，至此整个开发完成。对于webhook 的开发还是非常简单的，官方介绍的也非常详细。

下面我们通过 deployment 来创建两个pod，看看效果。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

```
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

这时我们再看一下webhook的打印日志

```
收到 kube-apiserver 发送的请求
v1.ObjectMeta{Name:"", GenerateName:"nginx-deployment-f6dc544c7-", Namespace:"default", SelfLink:"", UID:"", ResourceVersion:"", Generation:0, CreationTimestamp:time.Date(1, time.January, 1, 0, 0, 0, 0, time.UTC), DeletionTimestamp:<nil>, DeletionGracePeriodSeconds:(*int64)(nil), Labels:map[string]string{"app":"nginx", "pod-template-hash":"f6dc544c7"}, Annotations:map[string]string{"mutate":"云原生k8s-apiserver-webhook"}, OwnerReferences:[]v1.OwnerReference{v1.OwnerReference{APIVersion:"apps/v1", Kind:"ReplicaSet", Name:"nginx-deployment-f6dc544c7", UID:"373afa36-640d-481d-90c1-36f24dfcb919", Controller:(*bool)(0xc00003a1aa), BlockOwnerDeletion:(*bool)(0xc00003a1ab)}}, Finalizers:[]string(nil), ManagedFields:[]v1.ManagedFieldsEntry{v1.ManagedFieldsEntry{Manager:"kube-controller-manager", Operation:"Update", APIVersion:"v1", Time:time.Date(2023, time.August, 3, 20, 55, 34, 0, time.Local), FieldsType:"FieldsV1", FieldsV1:(*v1.FieldsV1)(0xc000010180), Subresource:""}}}
[default/]资源变更成功

收到 kube-apiserver 发送的请求
v1.ObjectMeta{Name:"", GenerateName:"nginx-deployment-f6dc544c7-", Namespace:"default", SelfLink:"", UID:"", ResourceVersion:"", Generation:0, CreationTimestamp:time.Date(1, time.January, 1, 0, 0, 0, 0, time.UTC), DeletionTimestamp:<nil>, DeletionGracePeriodSeconds:(*int64)(nil), Labels:map[string]string{"app":"nginx", "pod-template-hash":"f6dc544c7"}, Annotations:map[string]string{"mutate":"云原生k8s-apiserver-webhook"}, OwnerReferences:[]v1.OwnerReference{v1.OwnerReference{APIVersion:"apps/v1", Kind:"ReplicaSet", Name:"nginx-deployment-f6dc544c7", UID:"373afa36-640d-481d-90c1-36f24dfcb919", Controller:(*bool)(0xc00003a48a), BlockOwnerDeletion:(*bool)(0xc00003a48b)}}, Finalizers:[]string(nil), ManagedFields:[]v1.ManagedFieldsEntry{v1.ManagedFieldsEntry{Manager:"kube-controller-manager", Operation:"Update", APIVersion:"v1", Time:time.Date(2023, time.August, 3, 20, 55, 34, 0, time.Local), FieldsType:"FieldsV1", FieldsV1:(*v1.FieldsV1)(0xc000010498), Subresource:""}}}
[default/]资源变更成功
```

可以看到每当创建一个pod的时候，将调用一次webhook。

> 注意：上面两种方式打印日志输出有点不一要，通过 deployment 创建的pod的名称为空。那么为什么呢？

我们这里只开发了 `MutatingWebhook` ，对于`ValidatingAdmissionWebhook` 开发流程它们是完全一样的，上面的示例非常的简单，建议参考官方示例 [https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go](https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go)

> 这里的证书目前完全手动手动指定了， 不但麻烦，而且还有安全问题，那么有没有更好的解决办法呢？certmanager? configMap/secret[ [示例](https://github.com/ContainerSolutions/go-validation-admission-controller/tree/master)] ?

本文代码来自 [https://mp.weixin.qq.com/s/HszLzrYFfVk45_-Rq72HOA](https://mp.weixin.qq.com/s/HszLzrYFfVk45_-Rq72HOA) , 并进行了一些调整和bug修正。生产环境开发时建议使用官方仓库示例代码 [https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go](https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go)

# 参考资料 

 * [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/)
 * [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers)
 * [https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#validatingwebhookconfiguration-v1-admissionregistration-k8s-io](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#validatingwebhookconfiguration-v1-admissionregistration-k8s-io)
 * [https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go](https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go)
 * [https://mp.weixin.qq.com/s/HszLzrYFfVk45_-Rq72HOA](https://mp.weixin.qq.com/s/HszLzrYFfVk45_-Rq72HOA)
 * [https://zhuanlan.zhihu.com/p/487618315](https://zhuanlan.zhihu.com/p/487618315)
 * [https://blog.container-solutions.com/a-gentle-intro-to-validation-admission-webhooks-in-kubernetes](https://blog.container-solutions.com/a-gentle-intro-to-validation-admission-webhooks-in-kubernetes)

 [1]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#what-are-they
 [2]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do
 [3]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks
 [4]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/
 [5]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook
 [6]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook
 [7]: https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#failure-policy
 [8]: https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names
 [9]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#status-v1-meta
 [10]: https://jsonpatch.com/