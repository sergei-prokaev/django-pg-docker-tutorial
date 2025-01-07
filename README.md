# Настройка GitLab CI/CD и интеграция Kubernetes

В этом руководстве описаны шаги по настройке переменных в GitLab CI/CD и созданию `kubeconfig`, готового для использования в GitLab.

---

## Шаг 1: Настройте переменные в GitLab CI/CD

Перейдите в **Settings** → **CI/CD** → **Variables** и добавьте следующие переменные:

- `CI_REGISTRY`: URL вашего Docker Registry (например, `registry.gitlab.com`).
- `CI_REGISTRY_USER`: Логин для авторизации в Docker Registry.
- `CI_REGISTRY_PASSWORD`: Пароль или токен доступа.

---

## Шаг 2: Как создать kubeconfig для GitLab?

Следуйте инструкциям ниже, чтобы создать `kubeconfig` файл для GitLab.

### 2.1 Создание долгоживущего токена

Создайте секрет в кластере Kubernetes, используя команду:

```bash
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

Шаг 4. Проверка конфигурации

kubectl --kubeconfig gitlab-admin.conf get pods

3. Добавление kubeconfig в GitLab
Загрузите файл gitlab-admin.conf в Settings → CI/CD → Kubernetes cluster integration.
Либо добавьте его как переменную CI/CD типа File с именем KUBECONFIG.
Важные замечания
Права cluster-admin: созданный kubeconfig обладает правами cluster-admin — используйте его осторожно.
Рекомендации по безопасности:
Создайте отдельный namespace для GitLab.
Ограничьте права только этим namespace.
В продакшене: вместо cluster-admin рекомендуется использовать более ограниченные права.
Теперь вы успешно настроили CI/CD и интеграцию Kubernetes для GitLab!
