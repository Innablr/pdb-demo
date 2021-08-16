# A demo on `PodDisruptionBudget`

This is a very simple project to demonstrate the capability of `PodDisruptionBudget`.

## Prerequisite

You will need to prepare a cluster beforehand. The cluster can be an actual k8s cluster running in the cloud, or a `minikube`/`k3s`/`kind` cluster. The cluster should at least have **2** nodes. 

## Usage

1. Apply `manifests/deployment.yaml`
2. You should have two `nginx` pods running. 
3. Use `kubectl get pod -o wide` to verify if they are scheduled on the same node. 
   1. They should be. 
4. Wait until both of them are `1/1`.
5. Open up another terminal, and run `watch kubectl get pod`. 
   1. You should be able to see a list of all running pods. 
   2. The list should be refreshing once every few seconds. 
6. Now, take note of the node (_node 1_ from now on) on which those two pods are scheduled. 
7. Do `kubectl drain {{_node 1_}}`.
   1. You may need to supply `--ignore-daemonsets --force`
8. Take notice of the result of `watch kubectl get pod` in another terminal.
   1. Both pods should restart and become `0/1` 
   2. If you try to reach the pods either through a service or by port-forwarding, you should receive `Connection reset.`
   3. After a while, those two pods should be respawned on the other node (_node 2_)
9. Now, recover the state of node by `kubectl undrain {{_node 1_}}`
10. `kubectl apply -f manifests/pdb.yaml`
11. Do `kubectl drain {{_node 2_}}`. 
12. You should see the effect of `PodDisruptionBudget` now. 