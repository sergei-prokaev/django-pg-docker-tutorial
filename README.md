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
