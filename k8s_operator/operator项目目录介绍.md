在使用 **Kubebuilder** 创建的 Kubernetes Operator 项目中，目录结构非常清晰、标准，便于开发和维护。以下是一个典型的 Operator 项目的目录结构说明：

---

## 📁 Operator 项目目录结构（使用 `kubebuilder init` 创建）

```
my-operator/
├── api/                    # 💡 自定义资源（CRD）定义目录
│   └── <group>/<version>/  # 例如 webapp/v1/
│       ├── <kind>_types.go # 定义 CR 的 Spec/Status 结构
│       ├── groupversion_info.go # 注册 group/version
│       └── zz_generated.deepcopy.go # 自动生成的 deepcopy 工具
├── bin/                    # controller-gen 和 kustomize 等工具二进制（可选）
├── config/                 # 💡 用于部署的 YAML 清单集合
│   ├── crd/                # 生成的 CRD 文件目录
│   │   └── bases/          # 最终的 CRD YAML（由 make manifests 生成）
│   ├── rbac/               # Role, RoleBinding, ServiceAccount 权限配置
│   ├── manager/            # Deployment 定义（运行 Operator 的 Pod）
│   ├── samples/            # 示例 CR YAML（用于测试）
│   ├── default/            # 默认 kustomization 设置
│   └── kustomization.yaml  # 顶级 Kustomize 文件（组合部署）
├── controllers/            # 💡 控制器逻辑目录（处理 Reconcile）
│   └── <kind>_controller.go # 每个 CRD 对应的控制器逻辑
├── Dockerfile              # 构建 Operator 镜像的 Dockerfile
├── go.mod / go.sum         # Go 模块依赖管理
├── main.go                 # 💡 Operator 程序主入口（注册控制器等）
├── Makefile                # 💡 常用命令封装（build, deploy, manifests 等）
└── PROJECT                 # Kubebuilder 项目元数据文件
```

---

## 🔍 重点说明各核心目录功能

| 目录 / 文件 | 作用 |
|-------------|------|
| `api/` | 定义自定义资源的 Go 结构（Spec、Status） |
| `controllers/` | 实现每个 CR 的控制器逻辑（Reconcile 方法） |
| `config/` | 所有部署、权限、CRD、CR 示例的 YAML |
| `main.go` | 主函数：注册控制器、CRD、Scheme |
| `Makefile` | 快捷命令，如 `make run`、`make deploy`、`make manifests` 等 |
| `Dockerfile` | 用于构建 Operator 镜像 |
| `PROJECT` | Kubebuilder 用来管理项目的配置文件（非代码） |

---

## ✅ 推荐开发流程配合目录：

| 操作 | 相关目录 |
|------|----------|
| 创建新 CR 类型 | `api/` 和 `controllers/` |
| 写业务逻辑 | `controllers/<kind>_controller.go` |
| 修改权限 | `config/rbac/role.yaml` |
| 运行测试 CR | `config/samples/<group>_<version>_<kind>.yaml` |
| 打包部署 | `Dockerfile`, `Makefile`, `config/` |

---

## 🚀 实际开发中最常改的文件：

- `api/<group>/<version>/<kind>_types.go`  
- `controllers/<kind>_controller.go`  
- `config/rbac/role.yaml`  
- `config/samples/...`  
- `main.go`（注册 Scheme 或做全局设置）

---

需要我为你生成一套结构清晰的空 Operator 项目模板，或者进一步讲讲哪个目录如何配合调试吗？