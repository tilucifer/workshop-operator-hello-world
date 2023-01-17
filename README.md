# workshop-operator-hello-world-gitpod
Empty repository to start the workshop

## GitPod integration

To open the workspace, simply click on the *Open in Gitpod* button, or use [this link](https://gitpod.io/#https://github.com/k8s-operator-workshop/workshop-operator-hello-world).

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/k8s-operator-workshop/workshop-operator-hello-world)

## Configuration

Follows the steps describe in the [attendees-instructions.md](https://github.com/k8s-operator-workshop/workshop-instructions/blob/main/attendees-instructions.md) to set the `KUBECONFIG` environment variable.

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
2022-12-13 10:48:50,480 INFO  [io.qua.ope.run.OperatorProducer] (Quarkus Main Thread) Quarkus Java Operator SDK extension 4.0.3 (commit: d88d41d on branch: d88d41d78baf198fa4e69d1205f9d19ee04d8c60) built on Thu Oct 06 20:26:39 UTC 2022

2022-12-13 10:48:50,483 WARN  [io.qua.ope.run.AppEventListener] (Quarkus Main Thread) No Reconciler implementation was found so the Operator was not started.
2022-12-13 10:48:50,558 INFO  [io.quarkus] (Quarkus Main Thread) workshop-operator-hello-world 0.0.1-SNAPSHOT on JVM (powered by Quarkus 2.13.1.Final) started in 3.707s. Listening on: http://localhost:8080
2022-12-13 10:48:50,560 INFO  [io.quarkus] (Quarkus Main Thread) Profile dev activated. Live Coding activated.
2022-12-13 10:48:50,561 INFO  [io.quarkus] (Quarkus Main Thread) Installed features: [cdi, kubernetes, kubernetes-client, micrometer, openshift-client, operator-sdk, smallrye-context-propagation, smallrye-health, vertx]
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
$ kubectl get crds helloworlds.operator.workshop.com -o json | jq '.spec.versions[0].schema.openAPIV3Schema.properties.spec'

{
  "properties": {
    "name": {
      "type": "string"
    }
  },
  "type": "object"
}
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