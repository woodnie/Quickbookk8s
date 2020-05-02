\[root@master ~\]\# kubectl create deployment wood-busybox-dep --image=busybox

deployment.apps/wood-busybox-dep created

\[root@master ~\]\# kubectl get deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE

wood-busybox-dep   0/1     0            0           35s



