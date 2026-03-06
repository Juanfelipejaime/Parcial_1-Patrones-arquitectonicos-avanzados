# README – Despliegue de Aplicación con Helm y ArgoCD

## Descripción del proyecto

Este proyecto implementa el despliegue de una aplicación full-stack en Kubernetes utilizando Helm y ArgoCD para la gestión GitOps.

La arquitectura está compuesta por tres componentes principales:

- **Base de datos:** PostgreSQL (chart oficial de Bitnami) /La base de datos se crea pero por errores en la conexion no se conecto al backend/
- **Backend:** API de prueba basada en `httpbin`
- **Frontend:** Servidor Nginx que consume el backend mediante JavaScript

El despliegue se gestiona mediante **Helm charts** y se sincroniza automáticamente en el clúster mediante **ArgoCD**.

---

# Arquitectura

```
            +----------------------+
            |        Ingress       |
            |----------------------|
            | /        → Frontend  |
            | /api/*   → Backend   |
            +----------+-----------+
                       |
           +-----------+-----------+
           |                       |
      +-----------+          +-------------+
      | Frontend  |          |  Backend    |
      |  nginx    |          |  httpbin    |
      +-----------+          +-------------+
                                   

                             +------------+
                             | PostgreSQL |
                             |  Bitnami   |
                             +------------+
```

---

# 1. Instalación manual con Helm

## Requisitos

- Kubernetes cluster
- Helm instalado
- kubectl configurado
- NGINX Ingress Controller instalado

---

## 1.1 Agregar repositorio de Bitnami

El chart utiliza PostgreSQL como dependencia.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## 1.2 Instalar el chart en entorno DEV

```bash
helm install pedido-app-dev charts/parcial1 \
  --namespace dev \
  --create-namespace \
  -f charts/parcial1/values-dev.yaml
```

---

## 1.3 Instalar el chart en entorno PROD

```bash
helm install pedido-app-prod charts/parcial1 \
  --namespace prod \
  --create-namespace \
  -f charts/parcial1/values-prod.yaml
```

---

## 1.4 Verificar despliegue

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
```

---

# 2. Configuración de ArgoCD

ArgoCD se utiliza para implementar un flujo donde el estado del clúster se sincroniza automáticamente desde el repositorio Git.


## 2.1 Aplicación DEV

Archivo:

```
environments/dev/application.yaml
```

Define una aplicación ArgoCD que despliega el chart usando `values-dev.yaml`.

```yaml
helm:
  valueFiles:
    - values-dev.yaml
```

Namespace destino:

```
dev
```

---

## 2.2 Aplicación PROD

Archivo:

```
environments/prod/application.yaml
```

Define una aplicación ArgoCD que despliega el chart usando `values-prod.yaml`.

```yaml
helm:
  valueFiles:
    - values-prod.yaml
```

Namespace destino:

```
prod
```

---

## 2.3 Sincronización automática

Las aplicaciones ArgoCD están configuradas con sincronización automática:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Esto significa que:

- Cada cambio en el repositorio Git
- provoca automáticamente una actualización del clúster.

---

## Endpoints de acceso

### Entorno DEV

### Entorno
El entorno de producción fue configurado para acceso directo mediante la IP pública/externa del Ingress Controller:

- `http://129.212.149.42/prod/`
- `http://129.212.149.42/prod-api/get`
```

Este endpoint devuelve una respuesta JSON desde el servicio `httpbin`.

---

# 4. Verificación del sistema

Comandos útiles:

### Ver pods

```bash
kubectl get pods -A
```

### Ver servicios

```bash
kubectl get svc -A
```

### Ver ingress

```bash
kubectl get ingress -A
```

### Ver aplicaciones ArgoCD

```bash
kubectl get applications -n argocd
```

---

# 5. Tecnologías utilizadas

- Kubernetes
- Helm
- ArgoCD
- PostgreSQL (Bitnami)
- Nginx
- httpbin