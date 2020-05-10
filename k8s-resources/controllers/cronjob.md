FEATURE STATE: Kubernetes v1.8 beta
A CronJob creates Jobs on a repeating schedule.

One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.

Caution:
All CronJob schedule: times are based on the timezone of the kube-controller-manager.

If your control plane runs the kube-controller-manager in Pods or bare containers, the timezone set for the kube-controller-manager container determines the timezone that the cron job controller uses.

When creating the manifest for a CronJob resource, make sure the name you provide is a valid DNS subdomain name. The name must be no longer than 52 characters. This is because the CronJob controller will automatically append 11 characters to the job name provided and there is a constraint that the maximum length of a Job name is no more than 63 characters.