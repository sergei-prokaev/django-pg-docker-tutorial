Шаги по настройке:
Настройте переменные в GitLab CI/CD:

Добавьте следующие переменные в разделе "Settings" → "CI/CD" → "Variables":
CI_REGISTRY: URL вашего Docker Registry (например, registry.gitlab.com).
CI_REGISTRY_USER: Логин для авторизации в Docker Registry.
CI_REGISTRY_PASSWORD: Пароль или токен доступа.

Как создать kubeconfig, готовый для использования в GitLab?


Я помогу создать kubeconfig файл для GitLab. Давайте сделаем это пошагово:

Сначала создадим долгоживущий токен:
bash

Copy
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: gitlab-token
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: ubuntu
type: kubernetes.io/service-account-token
EOF
Теперь получим данные для kubeconfig:
bash

Copy
# Получаем токен
TOKEN=$(kubectl -n kube-system get secret gitlab-token -o jsonpath='{.data.token}' | base64 -d)

# Получаем адрес API сервера
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

# Получаем CA сертификат
CA_CERT=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')
Создадим сам kubeconfig файл:
bash

Copy
cat << EOF > gitlab-admin.conf
apiVersion: v1
kind: Config
clusters:
- name: default
  cluster:
    certificate-authority-data: ${CA_CERT}
    server: ${APISERVER}
contexts:
- name: default
  context:
    cluster: default
    user: gitlab
    namespace: default
current-context: default
users:
- name: gitlab
  user:
    token: ${TOKEN}
EOF
Проверьте работоспособность конфига:
bash

Copy
kubectl --kubeconfig gitlab-admin.conf get pods
В GitLab этот файл нужно добавить в:
Settings → CI/CD → Kubernetes cluster integration
Или как переменную CI/CD типа File с именем KUBECONFIG
Важные замечания:

Созданный kubeconfig имеет права cluster-admin - используйте его осторожно
Рекомендуется создать отдельный namespace для GitLab и ограничить права только этим namespace
В продакшене лучше использовать более ограниченные права вместо cluster-admin