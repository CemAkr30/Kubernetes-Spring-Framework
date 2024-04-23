adım adım bir Kubernetes deployment oluşturmak ve bunu dış dünyaya açmak:

Kubernetes Deployment Oluşturma:
İlk olarak, Kubernetes'te bir deployment oluşturuyoruz. Bir deployment, belirli bir uygulamanın (ya da mikro-servisin) birden fazla replikasını yönetmek için kullanılır. Bu, hizmetinizi yüksek erişilebilirlik ve ölçeklenebilirlik sağlar. Bir deployment oluşturmak için kubectl create deployment komutunu kullanırız.


1.Code
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE

Bu komut, hello-world-rest-api adında bir deployment oluşturur ve Docker Hub'dan in28min/hello-world-rest-api imajını kullanarak bu deployment için bir container başlatır.
Kümenin Dışarıya Açılması (Expose Edilmesi):
Oluşturduğumuz deployment'i dış dünyaya açmak için bir hizmet (service) oluştururuz. Hizmetler, Kubernetes kümesindeki diğer kaynaklara erişimi kolaylaştırmak için kullanılır. Bu adımda, dış dünyaya açılacak olan hizmetin tipini ve hangi port üzerinden erişileceğini belirtiyoruz.


2.Code
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080

Bu komut, hello-world-rest-api adındaki deployment'i dış dünyaya açar. --type=LoadBalancer argümanı, Kubernetes'in bulutta bir yük dengeleyici oluşturmasını ve hizmeti dış dünyaya açmasını sağlar. --port=8080 argümanı ise hizmetin hangi port üzerinden erişileceğini belirtir.
Bu adımlar sonucunda, hello-world-rest-api adındaki uygulamanız Kubernetes kümesinde başlatılır ve dış dünyadan erişilebilir hale gelir. Artık bu uygulamaya Kubernetes yük dengeleyicisi aracılığıyla erişebilirsiniz, bu sayede uygulamanızın yüksek erişilebilirlik ve ölçeklenebilirliğinden yararlanabilirsiniz.


kubectl get pods -> Deployment sahip hizmetin container pod'dur
kubectl get describe <depl_id> -> o deployment veya pod hakkında detaylı bilgi verir
kubectl get logs <pod_id> -> o hizmetin container da ki run time durumunda ki logu 
kubectl get replicaset <depl_id> -> o depl hakkında replica sayısını verir
kubectl get pods -o wide -> o wide commandi detaylı olarak pods hakkında bilgi verir ip vs 
kubetl get events -> kubernetes event logları'dır. 

kubectl scale deployment hello-world-rest-api --replicas=3 -> Var olan deployment olan hello-world-rest-api hizmetinin replicasetini deployment yönetir 3 farklı şekilde container oluştur diye deployments iletilir. Ve 3 farklı pods oluşturulmuş olur o deployment göre.




Kubernetes deployment çalışma mantığı -> *** Kubernetes deployment ilgili hizmete bir version atıldığı zaman deployments replicasetlerini yönetir eğer ki o deployment image geçersiz bir image ise kubernetes v1 versiyon daki
hizmetleri kapatmadan ilk v2 kaldırmaya çalışır farklı replicaseti üzerinde eğer ki invalid image vs vs hatalar varsa bir önceki v1 replicasetindeki container hizmeti down edilmediği için kesintiye uğramadan olduğu gibi çalışmaktadır. Fakat geçerli bir image
ile deploy atıldığı zaman hizmete replicaset de hizmetler oluşturulur hata yok ise v1 de ki replicasetin hizmetleri yani pods lar down edilir. Çalışma mantığı budur. Aşağıda iki farklı comamand line göreceğiz biri hatalı olan image diğeri doğru olan


kubectl get replicaset -o widekubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:TEST

#hatalı bir image olduğu için geçersiz yani

kubectl get pods diyerek status durumlarını görmüş olacağız v2 nin ancak v1 hala running de olacaktır. ne zaman down edilir v2 hatasız çalıştığı loadbalancer edildiği zaman down edilir.

kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.2.RELEASE






kubectl get deployment currency-exchange -o yaml -> dediğimiz anda kubernetes tarafından oluşturulan yapılandırma dosyasını göreceğiz. Bu hizmet için.  Bir kopyasını almak içinkubectl get deployment -o yaml >> deployment.yamlAynılarını service içinde yapabiliriz

kubectl get service currency-exchange -o yaml >> service.yaml

kubectl apply -f deployment.yaml -> bu yapılandırma dosyasını ilgili deployment hizmeti için update edebiliriz. Replica sayısını vs artırabilriiz düzenleme yapabiliriz ve kubernetes bildirmemiz gerekiyor ilgili deployment yansıtasmı için bu command o görevi görmektedir.

kubectl diff -f deployment.yaml -> bir önceki yapılandırma ile yeni yapılandırma review etmektedir.




kubectl delete all -l app=currency-exchange -> bu hizmete dair herşeyi silecektir. 

kubectl get svc --watch -> servicesleri izler detaylı bir şekilde 


  # kubectl create configmap currency-conversion --from-literal=CURRENCY_EXCHANGE_URI=http://currency-exchange
  # kubectl get configmap currency-conversion -o yaml >> deployment-03-final.yaml



kubectl rollout history deployment currency-conversion -> dağıtım kaç kere ypaılıp yapılmadığına bakabilrizi.


kubectl logs -f <pod_id> -> bir podun runtime yani container içerisine girer logu izler



kubectl rollout undo deployment currency-exchange --to-revision=1 -> ilk revizyon alan deployment hizmetinin versiyonuna geri döndürür bu command line.






kubectl autoscale deployment currency-exchange --min=1 --max=3 --cpu-percent=5 -> otomatik olarak kubernetes tarafından yatay ölçeklendirilecektir
kubectl delete hpa currency-exchange-> autoscale silecektir.



Kubernetes'in mimarisini ve bileşenlerini anlatayım:

Kubernetes Mimarisindeki Ana Bileşenler:

Master Node:
Master node, Kubernetes kümesini yöneten ana düğümdür. Aşağıdaki ana bileşenleri içerir:
API Server: Kubernetes kümesini yönetmek için kullanılan merkezi API sunucusudur. Diğer bileşenler, kaynakları yönetmek veya değişiklikler yapmak için API sunucusuna istek gönderirler.
Controller Manager: Kümedeki farklı işlevleri denetleyen bir kontrolcüdür. Örneğin, replica setler, deployment'lar, daemon setler gibi farklı kaynak türlerini kontrol eder.
Scheduler: Yeni bir pod'un hangi çalışma düğümüne yerleştirileceğini belirler. Boşta olan düğümleri, kaynak taleplerini ve politikaları dikkate alarak uygun olanları seçer ve yeni pod'ları dağıtır.
etcd: Kubernetes kümesinin tutarlı durumunu saklayan dağıtılmış bir key-value deposudur. Tüm küme durumu, yapılandırma ve durum bilgileri etcd'de saklanır.
Worker Node:
Worker node'lar, uygulama çalıştıran ve kaynakları sağlayan düğümlerdir. Her worker node, aşağıdaki bileşenleri içerir:
Kubelet: Kubernetes ajanıdır ve master node tarafından verilen talimatlara göre pod'ları çalıştırır, durumunu izler ve sağlığını rapor eder.
Kube Proxy: Servislerin ağ trafiğini yönlendirir ve pod'lar arasında erişilebilirliği sağlar.
Container Runtime: Pod'ları çalıştırmak için kullanılan konteyner çalışma zamanıdır. Örneğin, Docker veya containerd gibi bir konteyner çalışma zamanı olabilir.
Çalışma Prensibi:

Kullanıcı veya otomasyon araçları, Kubernetes API'si aracılığıyla kaynaklar oluşturur, günceller veya siler.
API sunucusu, gelen istekleri doğrular ve uygun kontrolcülere ileterek istenen durumu sağlamak için uygun eylemleri başlatır.
Kontrolcüler, küme durumunu izler ve gerektiğinde uygun eylemleri başlatır.
Kubelet'ler, master node tarafından sağlanan talimatları izler ve pod'ları düğümlerde çalıştırır ve durumlarını rapor eder.
Bu şekilde, Kubernetes kümesi, kaynakları etkin bir şekilde yönetir ve uygulamaların yüksek düzeyde ölçeklenebilir, güvenilir ve erişilebilir olmasını sağlar.





Docker & Kubernetes Command Line Script




docker run -p 8080:8080 in28min/hello-world-rest-api:0.0.1.RELEASE

kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
kubectl scale deployment hello-world-rest-api --replicas=3
kubectl delete pod hello-world-rest-api-58ff5dd898-62l9d
kubectl autoscale deployment hello-world-rest-api --max=10 --cpu-percent=70
kubectl edit deployment hello-world-rest-api #minReadySeconds: 15
kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.2.RELEASE

gcloud container clusters get-credentials in28minutes-cluster --zone us-central1-a --project solid-course-258105
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
kubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:TEST
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.2.RELEASE
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get componentstatuses
kubectl get pods --all-namespaces

kubectl get events
kubectl get pods
kubectl get replicaset
kubectl get deployment
kubectl get service

kubectl get pods -o wide

kubectl explain pods
kubectl get pods -o wide

kubectl describe pod hello-world-rest-api-58ff5dd898-9trh2

kubectl get replicasets
kubectl get replicaset

kubectl scale deployment hello-world-rest-api --replicas=3
kubectl get pods
kubectl get replicaset
kubectl get events
kubectl get events --sort.by=.metadata.creationTimestamp

kubectl get rs
kubectl get rs -o wide
kubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:TEST
kubectl get rs -o wide
kubectl get pods
kubectl describe pod hello-world-rest-api-85995ddd5c-msjsm
kubectl get events --sort-by=.metadata.creationTimestamp

kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.2.RELEASE
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get pods -o wide
kubectl delete pod hello-world-rest-api-67c79fd44f-n6c7l
kubectl get pods -o wide
kubectl delete pod hello-world-rest-api-67c79fd44f-8bhdt

gcloud container clusters get-credentials in28minutes-cluster --zone us-central1-c --project solid-course-258105
docker login
docker push in28min/mmv2-currency-exchange-service:0.0.11-SNAPSHOT
docker push in28min/mmv2-currency-conversion-service:0.0.11-SNAPSHOT

kubectl create deployment currency-exchange --image=in28min/mmv2-currency-exchange-service:0.0.11-SNAPSHOT
kubectl expose deployment currency-exchange --type=LoadBalancer --port=8000
kubectl get svc
kubectl get services
kubectl get pods
kubectl get po
kubectl get replicaset
kubectl get rs
kubectl get all

kubectl create deployment currency-conversion --image=in28min/mmv2-currency-conversion-service:0.0.11-SNAPSHOT
kubectl expose deployment currency-conversion --type=LoadBalancer --port=8100

kubectl get svc --watch

kubectl get deployments

kubectl get deployment currency-exchange -o yaml >> deployment.yaml
kubectl get service currency-exchange -o yaml >> service.yaml

kubectl diff -f deployment.yaml
kubectl apply -f deployment.yaml

kubectl delete all -l app=currency-exchange
kubectl delete all -l app=currency-conversion

kubectl rollout history deployment currency-conversion
kubectl rollout history deployment currency-exchange
kubectl rollout undo deployment currency-exchange --to-revision=1

kubectl logs currency-exchange-9fc6f979b-2gmn8
kubectl logs -f currency-exchange-9fc6f979b-2gmn8

kubectl autoscale deployment currency-exchange --min=1 --max=3 --cpu-percent=5
kubectl get hpa

kubectl top pod
kubectl top nodes
kubectl get hpa
kubectl delete hpa currency-exchange

kubectl create configmap currency-conversion --from-literal=CURRENCY_EXCHANGE_URI=http://currency-exchange
kubectl get configmap

kubectl get configmap currency-conversion -o yaml >> configmap.yaml

watch -n 0.1 curl http://34.66.241.150:8100/currency-conversion-feign/from/USD/to/INR/quantity/10

docker push in28min/mmv2-currency-conversion-service:0.0.12-SNAPSHOT
docker push in28min/mmv2-currency-exchange-service:0.0.12-SNAPSHOT

