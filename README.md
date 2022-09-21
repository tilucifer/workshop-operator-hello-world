# workshop-operator-hello-world-gitpod
Empty repository to start the workshop

## GitPod integration

Before open the project in Gitpod make sure to have set your `K8S_CTK` variable with the base 64 encoded version of your Kubernetes config file (`config-k3s00X`) file: `cat config-k3s00X | base64`.

To open the workspace, simply click on the *Open in Gitpod* button, or use [this link](https://gitpod.io/#https://github.com/k8s-operator-workshop/workshop-operator-hello-world).

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/k8s-operator-workshop/workshop-operator-hello-world)

## Workshop steps

ℹ️ The complete source of the workshop can be found in the [workshop-operator-hello-world-solution](https://github.com/k8s-operator-workshop/workshop-operator-hello-world-solution) repository ℹ️

### Init the project

  - create the project using the operator-sdk CLI: `operator-sdk init --plugins quarkus --domain operator.workshop.com --project-name workshop-operator-hello-world`
  - the following tree structure must be created:
```bash
.
├── LICENSE
├── Makefile
├── pom.xml
├── PROJECT
├── README.md
└── src
    └── main
        ├── java
        └── resources
            └── application.properties
```
  - change the JOSDK Quarkus extension version and the Quarkus version in the `pom.xml`:
```xml
<quarkus-sdk.version>4.0.1</quarkus-sdk.version>
<quarkus.version>2.12.2.Final</quarkus.version>
```
  remove the following dependency in the `pom.xml`:
```xml
quarkus-operator-sdk-csv-generator
```
  - add these dependencies in `pom.xml` for k3s compatibility:
```xml
    <!-- Mandatory for k3s : see https://github.com/fabric8io/kubernetes-client/issues/1796 -->
    <dependency>
      <groupId>org.bouncycastle</groupId>
      <artifactId>bcprov-ext-jdk15on</artifactId>
      <version>1.69</version>
    </dependency>
    <dependency>
      <groupId>org.bouncycastle</groupId>
      <artifactId>bcpkix-jdk15on</artifactId>
      <version>1.69</version>
    </dependency>
```
  - test the compilation: `mvn clean compile`
  - launch Quarkus in _dev mode_: `mvn quarkus:dev`:
```bash
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2022-09-16 07:13:33,717 WARN  [io.qua.config] (Quarkus Main Thread) Unrecognized configuration key "quarkus.operator-sdk.generate-csv" was provided; it will be ignored; verify that the dependency extension for this configuration is set or that you did not make a typo

2022-09-16 07:13:34,528 INFO  [io.qua.ope.run.OperatorProducer] (Quarkus Main Thread) Quarkus Java Operator SDK extension 4.0.1 (commit: 0a2f95e on branch: 0a2f95e4e591da2f562d7be0ee2039c6f83f3b47) built on Tue Sep 13 19:35:36 UTC 2022
2022-09-16 07:13:34,532 WARN  [io.qua.ope.run.AppEventListener] (Quarkus Main Thread) No Reconciler implementation was found so the Operator was not started.
2022-09-16 07:13:34,602 INFO  [io.quarkus] (Quarkus Main Thread) workshop-operator-hello-world 0.0.1-SNAPSHOT on JVM (powered by Quarkus 2.12.2.Final) started in 3.699s. Listening on: http://localhost:8080
2022-09-16 07:13:34,603 INFO  [io.quarkus] (Quarkus Main Thread) Profile dev activated. Live Coding activated.
2022-09-16 07:13:34,603 INFO  [io.quarkus] (Quarkus Main Thread) Installed features: [cdi, kubernetes, kubernetes-client, micrometer, openshift-client, operator-sdk, smallrye-context-propagation, smallrye-health, vertx]
```

### CRD generation
  - execute the following command: `operator-sdk create api --version v1 --kind HelloWorld`
  - check that the 4th classes had been generated:
```bash
src
│   └── main
│       ├── java
│       │   └── com
│       │       └── workshop
│       │           └── operator
│       │               ├── HelloWorld.java
│       │               ├── HelloWorldReconciler.java
│       │               ├── HelloWorldSpec.java
│       │               └── HelloWorldStatus.java
```
  - check the generated CRD in `./target/kubernetes/helloworlds.operator.workshop.com-v1.yml`
  - check that the CRD is generated in the Kubernetes' cluster: `kubectl get crds helloworlds.operator.workshop.com`
```bash
$ kubectl get crds helloworlds.operator.workshop.com

NAME                                CREATED AT
helloworlds.operator.workshop.com   2022-09-16T07:16:34Z
```
  - modify the `HelloWorldSpec.java` class:
```java
public class HelloWorldSpec {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
  - check that the CRD is updated in `./target/kubernetes/helloworlds.operator.workshop.com-v1.yml` file and in the Kubernetes' cluster:
```bash
$ kubectl get crds helloworlds.operator.workshop.com -o yaml

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  creationTimestamp: "2022-09-16T07:16:34Z"
  generation: 2
  name: helloworlds.operator.workshop.com
  resourceVersion: "35428210718"
  uid: 3dbbd774-cd57-4479-9c6b-5490a2d5362b
spec:
  conversion:
    strategy: None
  group: operator.workshop.com
  names:
    kind: HelloWorld
    listKind: HelloWorldList
    plural: helloworlds
    singular: helloworld
  scope: Namespaced
  versions:
  - name: v1
    schema:
      openAPIV3Schema:
        properties:
          spec:
            properties:
              name:
                type: string
            type: object
          status:
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: HelloWorld
    listKind: HelloWorldList
    plural: helloworlds
    singular: helloworld
  conditions:
  - lastTransitionTime: "2022-09-16T07:16:34Z"
    message: no conflicts found
    reason: NoConflicts
    status: "True"
    type: NamesAccepted
  - lastTransitionTime: "2022-09-16T07:16:34Z"
    message: the initial names have been accepted
    reason: InitialNamesAccepted
    status: "True"
    type: Established
  storedVersions:
  - v1
```
  - update the reconciler class `HelloWorldReconciler`:
```java
public class HelloWorldReconciler implements Reconciler<HelloWorld>, Cleaner<HelloWorld> { 
  private final KubernetesClient client;
  private static final Logger log = LoggerFactory.getLogger(HelloWorldReconciler.class);

  public HelloWorldReconciler(KubernetesClient client) {
    this.client = client;
  }

  @Override
  public UpdateControl<HelloWorld> reconcile(HelloWorld resource, Context context) {
    log.info("👋 Hello, World ! From {} 🌏", resource.getSpec().getName());

    return UpdateControl.noUpdate();
  }

  @Override
  public DeleteControl cleanup(HelloWorld resource, Context<HelloWorld> context) {
    log.info("🥲  Goodbye, World ! From {}", resource.getSpec().getName());
 
    return DeleteControl.defaultDelete();  
  }
}
```
  - create the namespace `test-hello-world`: `kubectl create ns test-hello-world`
  - create CR `./src/test/resources/cr-test-hello-world.yaml`:
```yaml
apiVersion: "operator.workshop.com/v1"
kind: HelloWorld
metadata:
  name: hello-world
spec:
  name: Moon
```
  - apply it on your namespace: `kubectl apply -f ./src/test/resources/cr-test-hello-world.yaml -n test-hello-world`
  - the logs in the console must display:
```bash
INFO  [com.wor.ope.HelloWorldReconciler] (EventHandler-helloworldreconciler) 👋 Hello, World ! From Moon 🌏
```
  - delete the CR: `kubectl delete helloworlds.operator.workshop.com hello-world -n test-hello-world`
  - the logs in the console must display:
```bash
INFO  [com.wor.ope.HelloWorldReconciler] (EventHandler-helloworldreconciler) 🥲 Goodbye, World ! From Moon
```