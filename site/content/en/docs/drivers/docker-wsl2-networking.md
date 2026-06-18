---
title: "Docker Driver on WSL2 - Networking Guide"
weight: 3.1
aliases:
    - /docs/reference/drivers/docker-wsl2
---

## Overview

When running Minikube with the Docker driver on Windows via WSL2, services deployed in the cluster are not directly accessible from your Windows browser. This is due to networking limitations between Windows host and the WSL2 virtual machine. This guide explains the cause and provides solutions.

## The Problem

### Why Can't I Access Services?

When you run `minikube start --driver=docker` in WSL2, Docker creates an internal bridge network (typically `172.17.0.0/16`). The Minikube cluster receives an IP address on this bridge, such as `172.17.0.2`.

**The Issue:** Windows cannot route traffic directly to Docker's internal bridge network inside WSL2. When you try to access a service using `minikube service` or the IP address in your Windows browser, the connection times out.

### Example

```bash
# In WSL2
minikube start --driver=docker
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080
minikube service hello-minikube
# Output: http://172.17.0.2:32498

# In Windows browser: Connection timeout - page won't load
```

## Solution 1: Using `minikube tunnel` (Recommended)

`minikube tunnel` creates a network route from your Windows host to the Minikube cluster, allowing direct access to services.

### Setup

1. Open a new PowerShell terminal on Windows (administrator privileges recommended):

```powershell
cd path\to\your\project
minikube tunnel
```

2. Leave this terminal running. It will display:

```
Status:	
	machine: minikube
	pid: 12345
	route: 10.96.0.0/12 -> 192.168.99.100
	minikube: Running
	services: []
```

3. In another terminal or your IDE, you can now access services using their cluster IP or service names (if you configure `/etc/hosts` or use DNS):

```bash
# Get the service IP
kubectl get svc hello-minikube
# Example output:
# NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
# hello-minikube    NodePort   10.96.12.34     <pending>     8080:32498/TCP

# Access via Windows browser
# http://10.96.12.34:8080
```

### Advantages
- Works for both NodePort and LoadBalancer services
- Persistent connection allows long-running services to remain accessible
- No additional configuration needed

### Disadvantages
- Requires keeping an additional terminal open
- May require administrator privileges on Windows

## Solution 2: Using `kubectl port-forward`

Port forwarding allows you to access a specific service by mapping a local port to the service port.

### Setup

1. In a WSL2 terminal, run:

```bash
kubectl port-forward svc/hello-minikube 8080:8080 --address=0.0.0.0
```

2. Access the service from your Windows browser:

```
http://localhost:8080
# or
http://127.0.0.1:8080
```

3. Or from WSL2 terminal:

```bash
curl http://localhost:8080
```

### Advantages
- No additional permissions required
- Can be configured per-service as needed
- Lightweight and simple for single service access

### Disadvantages
- Must be set up individually for each service
- Stops working if the terminal is closed
- Adds one additional port mapping step

## Solution 3: SSH Tunnel (Advanced)

For advanced scenarios, you can set up an SSH tunnel from Windows to WSL2.

```powershell
ssh -L 8080:172.17.0.2:32498 user@localhost
```

This maps local port 8080 to the Minikube service port. Access via `http://localhost:8080` in your Windows browser.

## Troubleshooting

### "Unable to connect" even with `minikube tunnel`

1. Verify the tunnel is running:
```bash
minikube tunnel status
```

2. Check that services are actually running:
```bash
kubectl get pods
kubectl get svc
```

3. Test connectivity from WSL2 terminal first:
```bash
curl http://10.96.12.34:8080
```

### Port conflicts

If you get a "port already in use" error:

```bash
# Find what's using the port (Windows)
netstat -ano | findstr :8080

# Kill the process
taskkill /PID <PID> /F
```

### WSL2 DNS resolution issues

If you're trying to use service names instead of IPs, ensure WSL2 DNS is working:

```bash
# In WSL2
nslookup hello-minikube.default.svc.cluster.local
```

## Best Practices

1. **For development:** Use `minikube tunnel` for continuous access to all services
2. **For testing single services:** Use `kubectl port-forward` to keep overhead minimal
3. **For production-like testing:** Combine both: use `minikube tunnel` and configure proper DNS/hosts entries
4. **Documentation:** Document your networking choice in your project's README for team consistency

## Related Resources

- [Minikube Service Documentation]({{<ref "/docs/commands/service">}})
- [Kubectl Port Forward Documentation](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
- [WSL2 Networking Guide]({{<ref "/docs/tutorials/wsl_docker_driver">}})
- [Docker Driver Setup]({{<ref "/docs/drivers/docker">}})
