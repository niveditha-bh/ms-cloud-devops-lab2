# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `ravi123/hello-app:v1`):

```
nivi1742/hello-app:v1
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
hello-app-6db684f4c-czj5g   1/1     Running   0          22h   10.244.1.5   desktop-worker    <none>           <none>
hello-app-6db684f4c-frqqk   1/1     Running   0          22h   10.244.2.6   desktop-worker2   <none>           <none>
hello-app-6db684f4c-tk4m9   1/1     Running   0          22h   10.244.1.4   desktop-worker    <none>           <none>
hello-app-6db684f4c-wpc7c   1/1     Running   0          21h   10.244.2.7   desktop-worker2   <none>           <none>
```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
PS C:\Users\Niveditha B H\ms-cloud-devops-lab2> kubectl get service hello-app
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.84.201   172.20.0.7    8080:31822/TCP   22h

PS C:\Users\Niveditha B H\ms-cloud-devops-lab2> kubectl get endpointslices -l kubernetes.io/service-name=hello-app
NAME              ADDRESSTYPE   PORTS   ENDPOINTS                                      AGE
hello-app-d7h77   IPv4          80      10.244.2.6,10.244.2.7,10.244.1.5 + 1 more...   22h
```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**
A Docker image built on my laptop is not automatically available on every Kubernetes node. Pushing the image to Docker Hub and referencing the correct image name allows Kubernetes to pull it.

**Trap 2 — the selector typo.**
Changing the Service selector from app: hello-app to app: helo-app caused the Service to lose its matching endpoints. The Pod labels and Service selector must match.

**Trap 3 — the wrong targetPort.**
Changing targetPort to 81 made the Service direct traffic to the wrong container port. The Pods were still discovered, but the application listens on port 80, so restoring targetPort: 80 fixed the routing.

**Trap 4 — the EXTERNAL-IP address.**
The LoadBalancer external IP reported by Docker Desktop was not directly accessible from my Windows browser. Using kubectl port-forward service/hello-app 8080:8080 allowed me to access the app at http://localhost:8080.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?

The generated Deployment YAML contained strategy and resources fields that were not explicitly specified in my manifest. Kubernetes supplies defaults for omitted configuration fields and manages runtime status, so my simpler manifest still works.
