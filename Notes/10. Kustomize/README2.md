# Kustomize Patching: Strategic Merge vs JSON 🔧

Quick reference for patching operations using one example deployment.

## Base Example 📋

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    app: nginx
    version: v1.0
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:1.20
          env:
            - name: ENV
              value: "dev"
            - name: DEBUG
              value: "false"
```

---

## 1. ADD Operations ➕

### Strategic Merge

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    environment: staging # Adds new label
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: NEW_VAR # Adds to env array
              value: "added"
```

### JSON Patch

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-app
    patch: |-
      - op: add
        path: /metadata/labels/environment
        value: staging
      - op: add
        path: /spec/template/spec/containers/0/env/-    # "-" means append to array
        value:
          name: NEW_VAR
          value: added
```

---

## 2. REMOVE Operations ➖

### Strategic Merge

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    version: null # null removes the field
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: DEBUG
              $patch: delete # $patch: delete removes from array
```

### JSON Patch

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-app
    patch: |-
      - op: remove
        path: /metadata/labels/version
      - op: remove
        path: /spec/template/spec/containers/0/env/1    # Must specify exact index
```

**🚨 Key Differences:**

- Strategic Merge: Use `null` for fields, `$patch: delete` for array items
- JSON Patch: Must know exact array index

---

## 3. REPLACE Operations 🔄

### Strategic Merge

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    version: v2.0 # Simply specify new value
spec:
  replicas: 5
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
```

### JSON Patch

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-app
    patch: |-
      - op: replace
        path: /metadata/labels/version
        value: v2.0
      - op: replace
        path: /spec/replicas
        value: 5
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: nginx:1.21
```

---

## 4. Array Manipulation Special Cases 📝

### Inserting at Specific Position

**Strategic Merge** (Limited control):

```yaml
# Can only append or replace entire array
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: ENV
              value: "dev"
            - name: NEW_FIRST # Will be merged/appended
              value: "value"
```

**JSON Patch** (Precise control):

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-app
    patch: |-
      - op: add
        path: /spec/template/spec/containers/0/env/1    # Insert at index 1
        value:
          name: NEW_MIDDLE
          value: inserted
```

**💡 Key Difference:** JSON Patch allows insertion at specific array positions

---

## 5. Conditional Array Operations 🎯

### Replace Array Item by Name (Strategic Merge)

```yaml
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: ENV # Matches by name, replaces value
              value: "production"
```

### Replace Array Item by Index (JSON Patch)

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-app
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/env/0/value    # Must know exact path
        value: production
```

**🔍 Key Difference:** Strategic Merge matches by key fields (like `name`), JSON Patch uses exact indices

---

## 6. Complex Array Scenarios 🏗️

### Adding Multiple Items

**Strategic Merge:**

```yaml
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: VAR1
              value: "val1"
            - name: VAR2
              value: "val2" # All items get merged
```

**JSON Patch:**

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-app
    patch: |-
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value: {name: VAR1, value: val1}
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value: {name: VAR2, value: val2}
```

---

## Quick Decision Guide 🤔

| Scenario             | Strategic Merge         | JSON Patch          |
| -------------------- | ----------------------- | ------------------- |
| Simple field changes | ✅ Easier               | ⚠️ More verbose     |
| Remove fields        | ✅ Use `null`           | ⚠️ Need exact path  |
| Remove from arrays   | ✅ Use `$patch: delete` | ⚠️ Need exact index |
| Array order matters  | ❌ Limited control      | ✅ Precise control  |
| Insert at position   | ❌ Can't do             | ✅ Specify index    |
| Match by key         | ✅ Natural behavior     | ❌ Must find index  |

**🎯 Quick Rule:** Use Strategic Merge for simple changes, JSON Patch when you need precise array control.
