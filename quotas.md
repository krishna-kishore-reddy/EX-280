## 🧩 1. Project Quotas

### 🔹 Concept

* A **ResourceQuota** limits the total resource consumption within a **project (namespace)**.
* It controls total:

  * CPU, memory, storage usage
  * Number of objects (Pods, Services, PVCs, etc.)
* Ensures fair resource sharing and prevents a single project from consuming everything.

---

### 🧠 Example Use Case

Limit a project to:

* Max 10 pods
* 5 PVCs
* 4 cores CPU total
* 8Gi memory total

---

### ⚙️ Declarative YAML (resourcequota.yaml)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: project-quota
spec:
  hard:
    pods: "10"
    persistentvolumeclaims: "5"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "6"
    limits.memory: 12Gi
```

**Apply it:**

```bash
oc apply -f resourcequota.yaml -n myproject
```

---

### 🖥️ Imperative Command

```bash
oc create quota project-quota \
  --hard=pods=10,persistentvolumeclaims=5,requests.cpu=4,requests.memory=8Gi,limits.cpu=6,limits.memory=12Gi \
  -n myproject
```

---

## 🧩 2. Cluster Resource Quotas

### 🔹 Concept

* A **ClusterResourceQuota** applies **across multiple projects (namespaces)**.
* Useful when you want to enforce quotas for a **group of projects** (like per team or per department).

---

### 🧠 Example Use Case

Limit all projects with label `team=dev` to:

* 50 pods total
* 20Gi memory total

---

### ⚙️ Declarative YAML (clusterresourcequota.yaml)

```yaml
apiVersion: quota.openshift.io/v1
kind: ClusterResourceQuota
metadata:
  name: dev-team-quota
spec:
  selector:
    annotations: {}
    labels:
      matchLabels:
        team: dev
  quota:
    hard:
      pods: "50"
      requests.memory: "20Gi"
      requests.cpu: "10"
```

**Apply it:**

```bash
oc apply -f clusterresourcequota.yaml
```

---

### 🖥️ Imperative Command

```bash
oc create clusterquota dev-team-quota \
  --project-label-selector=team=dev \
  --hard=pods=50,requests.memory=20Gi,requests.cpu=10
```

---

## 🧩 3. Limit Range

### 🔹 Concept

* A **LimitRange** sets **default, minimum, and maximum** compute resources for **containers and pods**.
* It prevents a user from creating unbounded resource requests.
* Applied **inside a project**.

---

### 🧠 Example Use Case

Enforce:

* Default CPU = 200m
* Default Memory = 256Mi
* Max CPU = 1 core
* Max Memory = 1Gi

---

### ⚙️ Declarative YAML (limitrange.yaml)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
spec:
  limits:
  - type: Container
    max:
      cpu: "1"
      memory: 1Gi
    default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
```

**Apply it:**

```bash
oc apply -f limitrange.yaml -n myproject
```

---

### 🖥️ Imperative Command

```bash
oc create limitrange resource-limits \
  --default-cpu=200m --default-memory=256Mi \
  --default-request-cpu=100m --default-request-memory=128Mi \
  --max-cpu=1 --max-memory=1Gi \
  -n myproject
```

---

## 🧩 4. Project Template

### 🔹 Concept

* A **Project Template** defines the default **objects automatically created** when a new project is created.

* Typically includes:

  * LimitRanges
  * ResourceQuotas
  * NetworkPolicies, etc.

* Set by cluster admins so that **every new project starts with the same baseline configuration**.

---

### ⚙️ Declarative YAML (project-template.yaml)

```yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: project-request-template
objects:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: default-quota
  spec:
    hard:
      pods: "10"
      requests.cpu: "2"
      requests.memory: "4Gi"
- apiVersion: v1
  kind: LimitRange
  metadata:
    name: default-limits
  spec:
    limits:
    - type: Container
      default:
        cpu: 200m
        memory: 256Mi
      max:
        cpu: 1
        memory: 512Mi
```

---

### 🖥️ Set it as the Default Project Template

```bash
oc apply -f project-template.yaml -n openshift-config
oc patch project.config.openshift.io cluster \
  --type=merge \
  -p '{"spec":{"projectRequestTemplate":{"name":"project-request-template"}}}'
```

Now every new project created by users will get the default quota and limitrange.

---

### 🧠 Example to Verify

```bash
oc get project.config.openshift.io cluster -o yaml | grep projectRequestTemplate
```

---

## 🧭 Summary Table

| Feature                  | Scope             | Resource Type          | Example Command                                | YAML Kind                  |
| ------------------------ | ----------------- | ---------------------- | ---------------------------------------------- | -------------------------- |
| **ResourceQuota**        | Per project       | `ResourceQuota`        | `oc create quota`                              | `v1`                       |
| **ClusterResourceQuota** | Multiple projects | `ClusterResourceQuota` | `oc create clusterquota`                       | `quota.openshift.io/v1`    |
| **LimitRange**           | Per project       | `LimitRange`           | `oc create limitrange`                         | `v1`                       |
| **Project Template**     | Cluster-wide      | `Template`             | `oc patch project.config.openshift.io cluster` | `template.openshift.io/v1` |

---

Would you like me to include **output samples** (like `oc describe quota`, `oc get limitrange`, etc.) to make it easier for your **EX280 practical review**?
