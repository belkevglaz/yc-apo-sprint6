### minikube 

```shell
sudo pacman -S minikube
```

Запустить minikube c метриками
```shell
minikube start --addons=metrics-server
```

Убедиться что metrics-server запущен
```shell
kubectl get deployments.apps --all-namespaces
```

Добавим Ingress
```shell
minikube addons enable ingress
```

Добавим в хосты
```shell
192.168.49.2  scalable.local
```

Развернем балалайку
```shell
kubectl apply -f ./scalable.yaml
```

Проверим что все ок
```shell
curl 'http://scalable.local/' --insecure                                                                                                                                                                                                                                                                   INT ✘ 
```

Ответ
```
Идентификатор пода: scalable-deployment-56cd59976c-wdhlv
```

### HPA

Развернем HPA
```shell
kubectl apply -f ./hpa.yaml
```

### Prometheus Operator

```shell
helm repo add prometheus-community <https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-operator prometheus-community/kube-prometheus-stack --values prometheus-values.yaml
```

Пробросим порт до Prometheus Web и проверим что есть метрики по нашим подам
```shell
kubectl port-forward service/prometheus-operated 9090:9090
```

Откроем в браузере [метрики](http://localhost:9090/graph?g0.expr=http_requests_total&g0.tab=1&g0.display_mode=lines&g0.show_exemplars=0&g0.range_input=6h).  
Если все ок, метрики есть - идем дальше

### Prometheus Adapter

Устанавливаем адаптер с нашим конфигом
```shell
helm install prometheus-adapter prometheus-community/prometheus-adapter -f prometheus-adapter-values.yaml
```

После старта проверяем что метрика есть
```shell
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

Вывод
```
{"kind":"APIResourceList","apiVersion":"v1","groupVersion":"custom.metrics.k8s.io/v1beta1","resources":[{"name":"namespaces/http_requests_per_second","singularName":"","namespaced":false,"kind":"MetricValueList","verbs":["get"]},{"name":"pods/http_requests_per_second","singularName":"","namespaced":true,"kind":"MetricValueList","verbs":["get"]}]}
```

Обновляем HPA
```shell
kubectl apply -f ./hpa-rps.yaml
```

Запускаем еще раз `locust`
