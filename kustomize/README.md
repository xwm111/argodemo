#### 使用kustomize 来管理argo-workflow  
kustomize  可以管理k8s的编排资源，argo-workflow也是一种k8s的crd资源，这里也可以通过kustomize来管理argo workflow .
kustomize 的目录结构如下：
```

├── base # 原始模板
│   ├── kustomization.yaml
│   └── whalesay-template.yaml #
├── deploy # 需要部署的环境
│   ├── dev # 开发
│   │   └── kustomization.yaml
│   ├── pro #生产
│   │   └── kustomization.yaml
│   └── test # test
│       └── kustomization.yaml
└── README.md
```
我们这里有原始的魔板base ，同时需要部署到三个不同的环境 开发环境，测试环境和生产环境。
我们使用kustomize 分别给dev加上dev的前缀 test环境加上test-的前缀,pro环境加上pro-的前缀。
base的配置如下：
```
#loony@26efeaec08d6:~/project/github/argodemo/kustomize$ cat base/whalesay-template.yaml 
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: workflow-template-whalesay-template
spec:
  templates:
  - name: whalesay-template
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```
这个是我们的魔板文件，现在通过定义kustomize的配置文件，base/kustomization.yaml 来定义一个kustomize的资源，方便其他地方进行引用
```
#loony@26efeaec08d6:~/project/github/argodemo/kustomize$ cat base/kustomization.yaml 
resources:
- whalesay-template.yaml
```
这样就定义了一个kustomize的资源。我们来看看我们定义的资源最终的输出是什么样子的. 
运行以下命令: kustomize build base/
```
#loony@26efeaec08d6:~/project/github/argodemo/kustomize$ kustomize build base/
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: workflow-template-whalesay-template
spec:
  templates:
  - container:
      args:
      - '{{inputs.parameters.message}}'
      command:
      - cowsay
      image: docker/whalesay
    inputs:
      parameters:
      - name: message
    name: whalesay-template
```
可以看到没有任何改变。
那么我们定义Dev的前缀名字:
```
# cat deploy/dev/kustomization.yaml 
bases:
- ../../base
namePrefix: dev-
```
然后看看定以后的资源最终的输出：
```
loony@26efeaec08d6:~/project/github/argodemo/kustomize$ kustomize build deploy/dev/
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: dev-workflow-template-whalesay-template
spec:
  templates:
  - container:
      args:
      - '{{inputs.parameters.message}}'
      command:
      - cowsay
      image: docker/whalesay
    inputs:
      parameters:
      - name: message
    name: whalesay-template
```
可以看到metadata.name发生了改变。其他的pro.test同理进行修改。
