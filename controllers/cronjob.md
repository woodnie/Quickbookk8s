###example

```
[root@master ~]# cat cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
[root@master ~]#


[root@master ~]# kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          92s

[root@master ~]# kubectl get job
NAME               COMPLETIONS   DURATION   AGE
hello-1590211380   0/1                      19m
hello-1590211860   0/1                      12m
hello-1590212340   0/1                      4m7s
hello-1590212400   0/1                      3m6s

[root@master ~]# kubectl get pod

[root@master ~]# kubectl logs hello-1590211380-mbkqq
Sat May 23 06:22:10 UTC 2020
Hello from the Kubernetes cluster


[root@master ~]# kubectl delete cronjob/hello
cronjob.batch "hello" deleted
```