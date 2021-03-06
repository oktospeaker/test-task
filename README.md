# test-task
Для деплоя RabbitMQ в k8s в первую очередь обратимся к официальной документации  - https://www.rabbitmq.com/clustering.html

Кластер RabbitMQ в k8s может быть сформирован используя механизм Kubernetes Peer Discovery Backend (начиная с версии 3.7.0), который поставляется в виде плагина в базовую поставку программного брокера сообщений. Репозиторий плагина на гитхабе - https://github.com/rabbitmq/rabbitmq-peer-discovery-k8s

Далее формируем конфигурационные файлы, ссылаясь на рекомендации и официальную документацию от RabbitMQ, примеры из репозитория плагина, а также на описанный опыт деплоя:

https://www.rabbitmq.com/cluster-formation.html#peer-discovery-k8s

https://github.com/rabbitmq/rabbitmq-peer-discovery-k8s

https://habr.com/ru/company/true_engineering/blog/419817/


<b>namespace.yaml</b> – Определяем пространство имен

<b>rbac.yaml</b> - Права доступа для RabbitMQ. Создаём ServiceAccount с правами на чтение Endpoints K8s.

<b>pv.yaml</b> – Хранилище данных. Взял самый простой случай — hostPath.

<b>pvc.yaml</b> – Создаем Volume Claim на томе, созданном в pv.yaml. Этот Claim затем будет использоваться в StatefulSet как хранилище постоянных данных.

<b>service.yaml</b> - Создаём внутренний headless сервис, через который будет работать Peer Discovery plugin.

<b>services.yaml</b> - Для работы приложений в K8s с нашим кластером создаём сервис балансировщика. RabbitMQ будет доступен при обращении к любой ноде кластера k8s по портам 31672 и 30672.

<b>configmap.yaml</b>, <b>statefulset.yaml</b> – Конфиги RabbitMQ

Далее запуск по мануалу:

https://github.com/rabbitmq/rabbitmq-peer-discovery-k8s/tree/master/examples/minikube#running-the-example-manually-with-minikube

После развертывания кластера необходимо приступить к беспростойной миграции данных из существующего нагруженного менеджера очередей RabbitMQ. Но это уже другая история.

Используем последнюю версию RabbitMQ 3.8.5 (от 15 июня 2020 года). Поддержка плагина rabbitmq-peer-discovery-k8s заявлена в этой версии. Требования к версии Kubernetes в документации не указано. Предполагаю совместимость с последней версией k8s от 15 июля 2020 года 1.18.6. Также из документации следуют следующие требования:

•	Minikube with the VirtualBox driver (или другие драйверы)

•	Kubernetes 1.6 (используется в примере на Гитхабе, но, предполагаю, подойдет и более новая версия)

•	RabbitMQ Docker image (официальный образ)

•	A StatefulSets controller

Выбрал этот путь развертывания, потому что в официальной документации к RabbitMQ есть отсылка к нему. Гугление показало, что метод действенный. Для более глубокой проработки задачи нужно больше входных данных, тестовый стенд и время.
