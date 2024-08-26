# Ceph CSI Log Rotate Design Document

Log rotation involves managing log files by controlling their size. This feature will be added to the CephCSI and csi-addons containers in the CephCSI node-plugin and controller-plugin pods. When logrotate is used, it will generate a log file at the host path and run the logrotator sidecar container in CephCSI pods. The logrotator sidecar container will then ensure that log files are rotated based on predefined settings.

Logroate configuration, 

`OperatorConfig CRD`:

```yaml
kind: OperatorConfig 
apiVersion: csi.ceph.io/v1alpha1
….
spec: 
    log:
        verbosity: 1 
    driverSpecDefaults:
        log:
            verbosity: 5
            rotation:
                # one of: hourly, daily, weekly, monthly
                periodicity: daily
                maxLogSize: 500M 
                maxFiles: 7
                logHostPath: /var/lib/cephcsi 
```

Similar settings will be overridden by `Driver CRD`:

```yaml
kind: Driver 
apiVersion: csi.ceph.io/v1alpha1 
metadata: 
    name: "<prefix>.<driver_type>.csi.ceph.com" 
    namespace:  <operator-namespace> 
spec: 
    log:
        verbosity: 1 
    driverSpecDefaults:
        log: 
            verbosity: 5
            rotation:
                 # one of: hourly, daily, weekly, monthly
                periodicity: daily
                maxLogSize: 500M 
                maxFiles: 7
                logHostPath: /var/lib/cephcsi 
```

Logrotator sidecar container cpu and memory usage can configured by,

`OperatorConfig CRD`:
```yaml
spec:
    driverSpecDefaults:
        controllerPlugin:
            resources:
                logRotator:
                    cpu: "100m"
                    memory: "32Mi"
        nodePlugin:
            resources:
                logRotator:
                    cpu: "100m"
                    memory: "32Mi"                          
```

For systems where SELinux is enabled (e.g. OpenShift), start plugin-controller as privileged that mount a host path.
`OperatorConfig CRD`:
```yaml
spec:
    driverSpecDefaults:
        controllerPlugin:
            privileged: true
```
