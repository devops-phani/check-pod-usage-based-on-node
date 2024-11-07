# check-pod-usage-based-on-node

Run remote script from curl.

```
curl -sL  https://raw.githubusercontent.com/devops-phani/check-pod-usage-based-on-node/refs/heads/main/ktop.sh | bash -s -- node1
```

Add the node name at the end of the line and execute it.

```
curl -sL  https://raw.githubusercontent.com/devops-phani/check-pod-usage-based-on-node/refs/heads/main/ktop.sh | bash -s -- 
```

```
vim ktop
```

```
#!/bin/bash

# Ensure a node name is provided as an argument
if [ -z "$1" ]; then
  echo "Usage: $0 <node-name>"
  exit 1
fi

# Print table header
printf "%-30s %-50s %-10s %-10s\n" "Namespace" "Pod" "CPU(%)" "Memory(%)"

# Retrieve the list of namespaces and pod names on the specified node
kubectl get pods --all-namespaces --field-selector spec.nodeName=$1 -o jsonpath='{range .items[*]}{.metadata.namespace}{" "}{.metadata.name}{"\n"}{end}' | 
while IFS= read -r ns_pod; do
    ns=$(echo "$ns_pod" | cut -d' ' -f1)
    pod=$(echo "$ns_pod" | cut -d' ' -f2)
    
    # Check if both namespace and pod name are present
    if [ -n "$ns" ] && [ -n "$pod" ]; then
        # Get resource usage using kubectl top pod
        metrics=$(kubectl top pod "$pod" --namespace="$ns" --no-headers 2>/dev/null)
        
        if [ -n "$metrics" ]; then
            # Extract CPU and memory usage from metrics
            cpu=$(echo "$metrics" | awk '{print $2}')
            memory=$(echo "$metrics" | awk '{print $3}')
            
            # Print row in the table
            printf "%-30s %-50s %-10s %-10s\n" "$ns" "$pod" "$cpu" "$memory"
        else
            printf "%-30s %-50s %-10s %-10s\n" "$ns" "$pod" "N/A" "N/A"
        fi
    else
        echo "Warning: Could not parse line: $ns_pod"
    fi
done

```

Provide execute permissions to script

```
chmod +x ktop
```

Check the pods usage in node1

```
./ktop node1
```

