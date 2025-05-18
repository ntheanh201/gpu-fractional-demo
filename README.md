# Fractional GPU Demo (HAMi)

This is a demo for fractional GPU on HAMi.

## Prerequisites

- Helm version v3+
- kubectl version v1.16+
- CUDA version v10.2+
- NvidiaDriver v440+

## Installation

- Docs: https://project-hami.io/docs/get-started/nginx-example

1. Install nvidia-container-toolkit
2. Label your nodes
    ```bash
    kubectl label nodes {nodeid} gpu=on
    ```

3. Deploy HAMi using helm
  
    ```bash
    helm repo add hami-charts https://project-hami.github.io/HAMi/
    helm install hami hami-charts/hami --set scheduler.kubeScheduler.imageTag=v1.16.8 -n kube-system
    ```

If everything goes well, you will see both vgpu-device-plugin and vgpu-scheduler pods are in the Running state

## Demo

1. Submit demo task

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: ubuntu-container
      image: ubuntu:18.04
      command: ["bash", "-c", "sleep 86400"]
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 vGPUs
          nvidia.com/gpumem: 3000 # Each vGPU contains 3000m device memory （Optional,Integer）
```

Verify in container resouce control:

```bash  
kubectl exec -it gpu-pod -- nvidia-smi
```

2. The result should be

```bash
[HAMI-core Msg(27:132256720885568:libvgpu.c:837)]: Initializing.....
Sun May 18 10:08:17 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.163.01             Driver Version: 550.163.01     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 2080        Off |   00000000:01:00.0 Off |                  N/A |
| 18%   40C    P8              1W /  215W |       0MiB /   3000MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
+-----------------------------------------------------------------------------------------+
[HAMI-core Msg(27:132256720885568:multiprocess_memory_limit.c:498)]: Calling exit handler 27
```
