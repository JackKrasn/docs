# Настройка cert-manager в k8s

## Для чего нужен cert manager

> cert-manager is a native Kubernetes certificate management controller. It can help with issuing certificates from a variety of sources, 
such as Let’s Encrypt, HashiCorp Vault, a simple signing keypair, or self signed.

**Cert manager** - это сервис в кластере, который следит за сертификатами. Когда у сертификата подходит время его жизни, 
то cert manager делает API запрос, например, к Let's Encrypt для получения нового сертификата и хранит его в secret. 

Проще говоря, не надо париться с сертификатами. Выпустил один CA, а дальше cert-manager будет следить за выпуском сертификатов, перевыпускать новые и т.д.

Например, необходимо развернуть менеджер репозиториев nexus и настроить ssl. Так вот при настроенном cert manager-e не придеться ничего делать с сертификатом.
Настроили Ingress и все, сертифкат будет выдан автоматически и за циклом его жизни тоже будет следить cert manager.  

Картинка с [официальной доки](https://cert-manager.readthedocs.io/en/latest/index.html):

![cert manager architecture](images/high-level-overview.png)

Я не буду использовать сервис Let's Encrypt, а поставлю свой собственный CA в secret и  с помощью него будут 
подписываться необходимые сертификаты.  


## Настройка

Необходимо выполнить следующую последовательность

### 1. Установка cert-manager с помощью helm
~~~
# helm install --name cert-manager --namespace kube-system stable/cert-manager
NAME:   cert-manager
LAST DEPLOYED: Fri Aug 10 10:09:03 2018
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/CustomResourceDefinition
NAME                               AGE
certificates.certmanager.k8s.io    1s
clusterissuers.certmanager.k8s.io  1s
issuers.certmanager.k8s.io         0s

==> v1beta1/ClusterRole
cert-manager  0s

==> v1beta1/ClusterRoleBinding
NAME          AGE
cert-manager  0s

==> v1beta1/Deployment
NAME          DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
cert-manager  1        1        1           0          0s

==> v1/Pod(related)
NAME                           READY  STATUS             RESTARTS  AGE
cert-manager-85545587cc-tvzh7  0/1    ContainerCreating  0         0s

==> v1/ServiceAccount
NAME          SECRETS  AGE
cert-manager  1        1s


NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.readthedocs.io/en/latest/reference/issuers.html

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.readthedocs.io/en/latest/reference/ingress-shim.html
~~~

Установка завершена

### 2.Настройка ClusterIssuer

Первым делом необходимо настроить ресурс **Issuer** или **ClusterIssuer**.

> These represent a certificate authority from which signed x509 certificates can be obtained, such as Let’s Encrypt, or your own signing key pair stored in a Kubernetes Secret resource.

> An Issuer is scoped to a single namespace, and can only fulfill Certificate resources within its own namespace. This is useful in a multi-tenant environment where multiple teams or independent parties operate within a single cluster.

> On the other hand, a ClusterIssuer is a cluster wide version of an Issuer. It is able to be referenced by Certificate resources in any namespace.

Для начала я создам **ClusterIssuer** и буду подписывать сертификаты своим собственным CA.  **ClusterIssuer** может выпускать сертификаты для сервисов в любом namespace. **Issuer** потребуется
создавать в каждом namespace.  

Поскольку я планирую подписывать все сертификаты одним CA, то мне нет смысла создавать в каждом namespace отдельный Issuer.


#### 1.Создать CA

Для тестов я создал самоподписанный сертифкат.
~~~
# Generate a CA private key
$ openssl genrsa -out ca.key 2048

# Create a self signed Certificate, valid for 10yrs with the 'signing' option set
$ openssl req -x509 -new -nodes -key ca.key -subj "/CN=${COMMON_NAME}" -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt
~~~

У меня сформировались 2 файла: ca.key, ca.crt.

#### 2. Сохранить сертификат и ключ в Secret

Для того, чтобы сохранить сертифкат и ключ в namespace **kube-system**  необходимо выполнить:
~~~
#kubectl create secret tls ca-key-pair  --cert=ca.crt --key=ca.key --namespace=kube-system
~~~

**Secret необходимо создать в namespace, в котором запущен pod cert-manager-а, а он запущен в namespace kube-system, о чем явно написано в документации**

> When referencing a Secret resource in ClusterIssuer resources (eg apiKeySecretRef) the Secret needs to be in the same namespace as the cert-manager controller pod. You can optionally override this by using the --cluster-resource-namespace argument to the controller.

Если захочется создать secret из файла, то помните что tls.cert и tls.key хранятся в base64. И как в этом случае созlать yaml файл описано тут [Generate TLS Secret for kubernetes](http://software.danielwatrous.com/generate-tls-secret-for-kubernetes/)

#### 3. Создать ClusterIssuer связанного с Secret ca-key-pair

Всегда читайте внимательно официальную доку [ClusterIssuers](http://docs.cert-manager.io/en/latest/reference/clusterissuers.html#clusterissuers)

Я создал ClusterIssuer из yaml файла следующего содержания:
~~~
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: ca-issuer
  #namespace: default
spec:
  ca:
    secretName: ca-key-pair
~~~

После этого необходимо проверить его статус, сэкономите кучу времени
~~~
$ kubectl get ClusterIssuer -o yaml
apiVersion: v1
items:
- apiVersion: certmanager.k8s.io/v1alpha1
  kind: ClusterIssuer
  metadata:
    clusterName: ""
    creationTimestamp: 2018-08-10T06:23:23Z
    generation: 1
    name: ca-issuer
    namespace: ""
    resourceVersion: "1833354"
    selfLink: /apis/certmanager.k8s.io/v1alpha1/clusterissuers/ca-issuer
    uid: e23b83ed-9c65-11e8-a198-0021f6aa4d96
  spec:
    ca:
      secretName: ca-key-pair
  status:
    conditions:
    - lastTransitionTime: 2018-08-10T06:34:03Z
      message: Signing CA verified
      reason: KeyPairVerified
      status: "True"
      type: Ready
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
~~~ 

Внимательно  посмотреть раздел status, чтобы там не было никаких ошибок. 

Например таких:
~~~
status:                                                                                                                                                                                                                                    
    conditions:                                                                                                                                                                                                                              
    - lastTransitionTime: 2018-08-10T05:40:17Z                                                                                                                                                                                               
      message: 'Error getting keypair for CA issuer: certificate is not a CA'
~~~

#### 4. Получить подписанный сертификат  

Ресурс типа **Certificate** необходимо создавать в том же namespace где находится сервис, для которого нужен подписанный сертификат.
Я приведу пример, как создавать ресурс явно, однако при правильной настройки **Ingress** этого не потребуется, т.к. сертификат будет 
выпускаться и подписываться cert manager-ом автоматически.

Создать Certificate из файла:

~~~
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: bs-kube
  namespace: sock-shop
spec:
  secretName: bs-kube-tls
  issuerRef:
    name: ca-issuer
    # We can reference ClusterIssuers by changing the kind here.
    # The default value is Issuer (i.e. a locally namespaced Issuer)
    kind: ClusterIssuer
  commonName: bs-kube.ftc.ru
  dnsNames:
  - bs-kube.ftc.ru
  - docker.ftc.ru
~~~

Можно прочитать тут [Obtain a signed Certificate](http://docs.cert-manager.io/en/latest/tutorials/ca/creating-ca-issuer.html#obtain-a-signed-certificate)

После создания сертификата убедиться, что успешно появился в кластере.
~~~
$kubectl describe certificate -n sock-shop bs-kube
Name:         bs-kube
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  certmanager.k8s.io/v1alpha1
Kind:         Certificate
Metadata:
  Cluster Name:
  Creation Timestamp:  2018-08-10T05:44:37Z
  Generation:          1
  Resource Version:    1825538
  Self Link:           /apis/certmanager.k8s.io/v1alpha1/namespaces/default/certificates/bs-kube
  UID:                 77a3974c-9c60-11e8-a198-0021f6aa4d96
Spec:
  Common Name:  bs-kube.ftc.ru
  Dns Names:
    bs-kube.ftc.ru
    docker.ftc.ru
  Issuer Ref:
    Kind:       Issuer
    Name:       ca-issuer
  Secret Name:  bs-kube-tls
Status:
  Conditions:
    Last Transition Time:  2018-08-10T05:44:39Z
    Message:               Certificate issued successfully
    Reason:                CertIssued
    Status:                True
    Type:                  Ready
Events:
  Type    Reason      Age   From          Message
  ----    ------      ----  ----          -------
  Normal  IssueCert   9s    cert-manager  Issuing certificate...
  Normal  CertIssued  9s    cert-manager  Certificate issued successfully
~~~

И посмотреть secret в котором хранится сам сертификат:
~~~
$kubectl get secret bs-kube-tls -n sock-shop -o yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURKakNDQWc2Z0F3SUJBZ0lSQU1WL1pMRGloUWZ0YmpyakJQUzgyTUl3RFFZSktvWklodmNOQVFFTEJRQXcKRHpFTk1Bc0dBMVVFQXd3RVluTmpZVEFlRncweE9EQTRNVEF3TnpJM01UQmFGdzB4T1RBNE1UQXdOekkzTVRCYQpNREF4RlRBVEJnTlZCQW9UREdObGNuUXRiV0Z1WVdkbGNqRVhNQlVHQTFVRUF4TU9Zbk10YTNWaVpTNW1kR011CmNuVXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFETVhnOGhZMkVkWDQra2FOVlUKeS9HSHlmQ0hFQXVUV2pvby9nK3FVemthTFQzdXJSb1ppMS81K1FFaml0eUxiQ056NFZ0cWY1WmhtOFJuVDVkZwp2TXhOV1ZKdENxMnVzZFZia0RVd1QwU1lBTnlrNmxpamM3WmVXZ0lXUjhnUE9ZamtPWmtPUHF3c05Mc3laYitoCk91VGdaOVF2eUo5S2p1UnowMHc0d2NCbXBYL0RpS2RpaEtsdkh3UDdTZnZla00wTHJHQzNCNVJmbUVsTXFSTHUKSS9oOE81NjlwV2NGOWhhRHFaL2dqN2VJYmtnWFhYR2RWOG9zYUdHbnhtZ1MrN3FBWUFpRk84RjlWeW5zQlYvMgpZZmZ3RmFKWWhmczJRdEsrOG1IQUsyS2JrU1N6SjhXb0xmSDB3emUydHlZY2c2NVlXbHhncVlpdUJyaW9uRXN4Ci8vUE5BZ01CQUFHalhEQmFNQTRHQTFVZER3RUIvd1FFQXdJRm9EQU1CZ05WSFJNQkFmOEVBakFBTUI4R0ExVWQKSXdRWU1CYUFGSHZTOTVLeHJ5bk9tWW5QeXBZY1ZBb2NqTGFnTUJrR0ExVWRFUVFTTUJDQ0RtSnpMV3QxWW1VdQpablJqTG5KMU1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQnlkcHp4UDVNLzhqYjd5dXQ2bmNleXR6ZVFsTTJQClJsWXJxa0s5bWVNUEp6K3NmM3JGV0pZME9ER3RMNzlNWktWc1JBc2lCSHI2Rmg3dkVNczRCQkJzbG1YVnFYS3kKaWNkVk1oazVQNWxmbWYyaXFCU1d5dnI3SXd5S0JCd3BMYTVhbll1eEJOaWpidjNvM2ZnbUFuQ0xQSzNPKzExVwpsTXRjd1d6NTFVamtwdE00MHpkWFc4TlhaUWxMSlZOcEFGRUo2S0VwRG1aQk84dWp0T1E3Yk8wR01OSEZkYll4CmxmVHlITnlMVGdpNktBVUZ1WkNrb1IvZGtqcWI4SFJvdjRMYjhHU1kyNWNQTGNTdU9JZUoyUDRQRk5sTHluK1QKYWEyVnhWVnJaSUpCWkt2TENoNkZwb0JQa1V0R1FFTy83cy9pLzBvdlhCUEJmWTR3SS8rR3RqcGMKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQotLS0tLUJFR0lOIENFUlRJRklDQVRFLS0tLS0KTUlJQzlEQ0NBZHlnQXdJQkFnSUpBSmxxaExpWWtrMmVNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1BOHhEVEFMQmdOVgpCQU1NQkdKelkyRXdIaGNOTVRnd09ERXdNRFUwTWpRMldoY05Namd3T0RBM01EVTBNalEyV2pBUE1RMHdDd1lEClZRUUREQVJpYzJOaE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBdFhTcDRLTjYKNU1UeCtNMEpHZjRLNkduWmJ1Zi9ZdHhQUjNIQnZQWmk1U2JyK0FUQTZNQW9UMzhCK1kyUERWWGVXSGtmTWt0QQpub2t3VzlkRWVBdzZtNUxOSXBDU3phT3dMUkw2RWFzM3ZwVk5TOWQ0YU9KMzh4dEVReVB2Vk1MdVlLVmUzRTZhCkZoQlZvTlhIZEt5SWU4WW5ZTVFWVlUyd0VUMmVjMkx3M3VvcjExcmEyY3JYajkya054VER3dFRZcVJJV0t3L3gKbHdtYTR2UCtLYVNDNlBqOFJYakhoQ0hlVUVySy90dVJzUUl3djFvOE93NDQ1aG02U0hhMlFlYUJ1QmxwKzN5eApYOVFTT1l0SXNiQzU2WFNkREdSbDFQdklsMkoyVmh1VURTWTk4a1BjcFRoanFuUFlzaXhXZzhjdGd2Mlh5TmhtCm1aS0Vld0thenZkK253SURBUUFCbzFNd1VUQWRCZ05WSFE0RUZnUVVlOUwza3JHdktjNlppYy9LbGh4VUNoeU0KdHFBd0h3WURWUjBqQkJnd0ZvQVVlOUwza3JHdktjNlppYy9LbGh4VUNoeU10cUF3RHdZRFZSMFRBUUgvQkFVdwpBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQWNjemovRkhSZm43S1NvOWxqRndFVmRoek90SVJCdkxtCjlaNmJuZjRvTUtiYzJUSlZZZ2V0QXhrc1pmU3FSbHhFa2VONUZoWDBWMklUd2d3MUNxZkt3YmdIZXpRODdtOG4Ka0p6MDNYQ2FGWjRzdmhHMzhZK1p6bFZEb1VMclJpeGtIanF6QW1WcUJiTExGMVVNbFRZWkRvRXVGaDl1c1kzNAo4UjRxNnVXNFNSeE16QVNzRGFEU0RuZWpzd1FSekU3MDI0WVBTSkViU09VQm9mZ1o5ak5rTlA1cmFyekZjYitaCjlQaXpnRlI5emR6TEU5aUhNRlBBV3NiNkI4R2RsSG5DWlVzaW5Ib3RLcEUxeUJwbjRUY1NmWjFycXR0SVJwZTcKSWJSL2wvSVExOFN6Umg3eEdKMzJoR1k0Ykx1MThDSjloS0lCZ0EwN2FxSlhFUG90S2ZnU0h3PT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBekY0UElXTmhIVitQcEdqVlZNdnhoOG53aHhBTGsxbzZLUDRQcWxNNUdpMDk3cTBhCkdZdGYrZmtCSTRyY2kyd2pjK0ZiYW4rV1ladkVaMCtYWUx6TVRWbFNiUXF0cnJIVlc1QTFNRTlFbUFEY3BPcFkKbzNPMlhsb0NGa2ZJRHptSTVEbVpEajZzTERTN01tVy9vVHJrNEdmVUw4aWZTbzdrYzlOTU9NSEFacVYvdzRpbgpZb1NwYng4RCswbjczcEROQzZ4Z3R3ZVVYNWhKVEtrUzdpUDRmRHVldmFWbkJmWVdnNm1mNEkrM2lHNUlGMTF4Cm5WZktMR2hocDhab0V2dTZnR0FJaFR2QmZWY3A3QVZmOW1IMzhCV2lXSVg3TmtMU3Z2Smh3Q3RpbTVFa3N5ZkYKcUMzeDlNTTN0cmNtSElPdVdGcGNZS21JcmdhNHFKeExNZi96elFJREFRQUJBb0lCQVFEQTVlczhKWlNONkJ3cQpJRFYwYzRmSUZzanNneTJaNlNsS2Rmd05WYjVwUWRqYVJ4T0NsdmFCZVJJbEhUWHNkNmJEQXl5SlNtS0VRVVhTCkNlTWxrUzc1dDF6QXhicUlVUnpFNzBuMURtejlXSnJySXJPRm5IdS9kUS9SUGZITXhRNjc5TTNPRDBQdCtkdlQKeHd4b3Y3RTNTMm1SckxrRjQvZ29oNEhEZE1ZSlcyMHV3bTBXaHAwa3R2cXNhSnhZOVI1ZTFuNzU5Y0lEZjY0Swp4L2p6OGtONnhheXpNWTVJL3BqQXVLNGFCVHY3L2hDeW4vb2VkdnYvdVR5WDV5MmRJN0twS1lMeVdWRkhXTGJYCjVoZkdTZ25FNXUwMGdCdmtzZTNYUXhSS1U2aFgybE15eVUrcDVNamh5RnJ0VUhQK0J1VUQxbldHQ0dsQ1hqY1EKNm9ENHVCT2hBb0dCQU00NkNHSUlBK1FjUlM5ckpvczZKVDZhL1o3OVdTWTVtb2RyV05DcXorRkZsdFY4NnIrcQpHN0FTekNma0toUXJFRkhrSVFIVDNOL3k1b1pKQWQvS2tTRWUya0xpL3dKS3hEZFVzRGE3TWptdUQ2aDV5SnVLCkcrWVErZHVBUTk1cDREMWFCS0xud1J3bVFmd0lXRUlYdGlrZUFhYnF2ZWZBS1RuNnJkV2Urd3JWQW9HQkFQMngKSmlMZ1dIMFJ0aGJRVkNOWnI2cGlGQ0R4NUF4TlYwREt1TUJpOVN4MzVlWUIzb1lYZW01QjBEZlhYblU2ZUxLRgpkTFZKUmx6eGllclRvbGkvVlZNQXpuQkJDM3J3ZitqMVhNMU9kdEliS3pOdkh0YzVnblhtdStrVkR5d2REZEhnCm90VjNJclZUWitQMzJ2NDZ6Q3pkYnE2MmdtZEJ1SFZHdmJ6akZORVpBb0dCQUxZNGFQZldCVG9tRUt2WmpmRXYKcTRFcUNqZlZ2RlFlU2dDbVJZLzduanQ2OWRBbDFIY09vL0JzYTZCRDV6cHk0clM1VXNEK3B3ZnE1TzU2ekFFbQpTQjV2MklPSmQ3SmF4ZzN0OHRZcGlqT1ZMWmk4SXhuc2FzSmE5YXVTSm1YOFAzdDJjdXBPeFQ5T1ByZW4xL1J1Clp6TGxwS2dNZTBpdmJyNGdWa0dQZkZzUkFvR0FVUUVGaWxGVUwrczkxeDhDSHA5K2hjcjNYbVdlU1lkUWV2Ry8KK0Q0Z3h1Z3AwajE2amhwbEQxdVlYcHc0SHZwaW02NGRTOTF2eURHZnRnbUpad2tBOTVYa1ZOZVFFTnRHSEY1cwpHV29hYXBBZVJUZ1FBdXpzQ1RWNWZyMG9zUTg5NEd2MzBtMU4rZFA5OGo2c0FFMUo4SEZyN0FGK3RmVzRMa28rClAxZkt4OUVDZ1lBVWFlejFydzhhUEJPRWE1OXdORnpCb25lTVVwaXNLUHlhSisvZXk0YUoycDdlN1E2VEpsVzgKMTFvMVIxZVM4bTBkeVNHKzQyc3lQRmhQZTAwcGxnVk00eUxqazYzS2d3cUhjakxKSzBDc2VyZHoxQXZ4MldDMQpGZ1grQXpiWlJOQWtteFp5UlRPMEc3TnFBcU93UDRGL2dEalBBUFlEam8ydmxyM2ZOZXJKS0E9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
kind: Secret
metadata:
  annotations:
    certmanager.k8s.io/alt-names: bs-kube.ftc.ru
    certmanager.k8s.io/common-name: bs-kube.ftc.ru
    certmanager.k8s.io/issuer-kind: ClusterIssuer
    certmanager.k8s.io/issuer-name: ca-issuer
  creationTimestamp: 2018-08-10T07:27:10Z
  labels:
    certmanager.k8s.io/certificate-name: bs-kube-tls
  name: bs-kube-tls
  namespace: sock-shop
  resourceVersion: "1841720"
  selfLink: /api/v1/namespaces/sock-shop/secrets/bs-kube-tls
  uid: caf9dbb8-9c6e-11e8-a198-0021f6aa4d96
type: kubernetes.io/tls
~~~

### Настрйока Ingress

Теперь необходимо настроить **Ingress** для доступа по https к развернутому приложению sock-shop.

~~~ yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: front-end-tls
  namespace: sock-shop
  annotations:
    #certmanager.k8s.io/cluster-issuer: ca-issuer
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - bs-kube.ftc.ru
    secretName: bs-kube-tls  
  rules:
  - host: bs-kube.ftc.ru
    http:
      paths:
      - path: /
        backend:
          serviceName: front-end
          servicePort: 80

~~~

Теперь sock-shop доступен по https.

А для того, чтобы сертифкат выпускался автоматически необходимо в **Ingress** указать аннотацию `certmanager.k8s.io/cluster-issuer`
и указать свой ClusterIssuer. В моем случае это ca-issuer.

Подробнее про аннотации [Supported annotations](http://docs.cert-manager.io/en/latest/reference/ingress-shim.html#supported-annotations)
и тут [Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)

В итоге я настроил так:
~~~ yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: front-end-tls
  namespace: sock-shop
  annotations:
   #kubernetes.io/tls-acme: "true"
    certmanager.k8s.io/cluster-issuer: ca-issuer
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - bs-kube.ftc.ru
    secretName: bs-kube-tls
  rules:
  - host: bs-kube.ftc.ru
    http:
      paths:
      - path: /
        backend:
          serviceName: front-end
          servicePort: 80
~~~


Собственно и все. 
~~~
$kubectl get ing -n sock-shop front-end-tls 
NAME            HOSTS            ADDRESS   PORTS     AGE
front-end-tls   bs-kube.ftc.ru             80, 443   1h
~~~