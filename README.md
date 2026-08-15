# strimzi-keda

Entorno

Windows 11
Docker Desktop
Minikube 1.38.1
Kubernetes 1.32.2
Helm 3.20.1
8 GB de RAM
IntelliJ

objetivo principal: KEDA escala consumidores de Kafka

1 Crear Minikube
minikube start --driver=docker --cpus=2 --memory=3076 --disk-size=20g

2 Crear namespace
kubectl create namespace kafka

3 Instalar Strimzi
helm repo add strimzi https://strimzi.io/charts/
helm repo update

helm install strimzi-kafka-operator strimzi/strimzi-kafka-operator `
  --namespace kafka `
  --version 0.45.2

4 Aplicar Kafka
kubectl apply -f lab-keda/02-kafka.yaml

Deberíamos terminar con aproximadamente:

edu-cluster-kafka-0                  1/1   Running
edu-cluster-zookeeper-0              1/1   Running
edu-cluster-entity-operator-xxxxx    3/3   Running
strimzi-cluster-operator-xxxxx       1/1   Running

5 Crear Topic
kubectl apply -f lab-keda/03-topic.yaml

6 Crear el Consumer
kubectl apply -f lab-keda/04-consumer.yaml

7 Instalar KEDA

helm repo add kedacore https://kedacore.github.io/charts
helm repo update

helm install keda kedacore/keda `
  --namespace keda `
  --create-namespace

Esperamos pods como:

keda-operator-...
keda-metrics-apiserver-...
keda-admission-webhooks-...

8 Crear el ScaledObject
kubectl apply -f lab-keda/05-keda.yaml

9 Crear Producer (Vamos a usar la misma imagen de Kafka que utiliza nuestro Consumer: quay.io/strimzi/kafka:0.45.2-kafka-3.8.0)

kubectl run my-producer --image=quay.io/strimzi/kafka:0.45.2-kafka-3.8.0 --restart=Never -n kafka --command -- /bin/bash -c 'for i in {1..100}; do echo message-$i; done | bin/kafka-console-producer.sh --bootstrap-server edu-cluster-kafka-bootstrap:9092 --topic scaling-demo'

Esto genera:

message-1
message-2
message-3
...
message-100

y los envía a: scaling-demo, Después el Pod termina.