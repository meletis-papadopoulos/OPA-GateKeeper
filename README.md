# OPA/GateKeeper Policy
> Block all Deployments/StatefulSets if a HPA doesn't exist!

## Description
> The project consists of 3 folders:
- **manifest** -> YAML files for resources
- **scripts** -> Scripts for clean up, load increase and Vim
- **ticket** -> OPA/Gatekeeper policy requirements

## How to run
> You need to have **root** privileges on the cluster

- Install Metrics Server (i.e. v0.6.3)
- Install Gatekeeper resources (use a pre-built image)
- Apply OPA Constrant Template
- Apply OPA Constraint
- Create test resources (i.e. Deployment/StatefulSet)
- Increase load (add more replicas with HPA)
- Clean Up!

## Remove applied resources
> Use **clean-up.sh** script

```bash
#!/bin/bash

cd /manifest

# Remove Deployment/Service/HPA
kubectl delete -f deployment-with-hpa.yaml --force --grace-period 0

printf "\n"

# Uninstall Gatekeeper Resources
kubectl delete -f gatekeeper.yaml --force -grace-period 0

printf "\n"

# Delete Constraint & Constraint Template
kubectl delete -f constraint.yaml --force --grace-period 0
kubectl delete -f constraint-template.yaml --force --grace-period 0

printf "\n"

# Remove Metrics Server
kubectl delete -f metrics-server.yaml --force --grace-period 0

# Check status
echo $?
```

## Override default Vim settings
> Use **custom-vim-settings.sh** script

```bash
#!/bin/bash

# Override default vim settings
VIM_PATH="/etc/vim/vimrc"

echo "set ai" >> ${VIM_PATH}
echo "set nu" >> ${VIM_PATH}
echo "set et" >> ${VIM_PATH}
echo "set sw=2" >> ${VIM_PATH}
echo "set ts=2" >> ${VIM_PATH}
```

## Increase workload
> Use **increase-load.sh** script

```bash
#!/bin/bash

# Run this in a separate terminal
# so that the load generation continues and you can carry on with the rest of the steps
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

## Links
Official HPA walkthrough [k8s docs](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
