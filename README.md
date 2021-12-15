# Kubernetes-log4j-check
A silly script that is checking every pod to see if "log4j" exists.

```bash
#!/bin/bash

for ns in $(sudo kubectl get ns | awk 'NR>1 {print $1}')
do
        for pod in $(sudo kubectl get pods -n $ns | grep Running | awk 'NR>1 {print $1}')
        do
                for  container in $(sudo kubectl describe pod $pod -n $ns | grep -v "Init Containers:" | sed -n -e '/Containers:/,$p' | grep -B 1 "Container ID": | grep -v init | grep -v "Container ID:" | grep -v "\-\-")
                do
                        echo "In $pod with ${container%?}: "
                        log4j=$(sudo kubectl exec -it $pod -n $ns -c ${container%?}  -- rpm -qa | grep log4j)
                        if [[ -n $log4j ]]; then
                                echo "FOUND log4j!!"
                                echo $log4j
                        else
                                echo "No log4j was found!"
                        fi

                        echo "========================================="
                done

        done
done
```
