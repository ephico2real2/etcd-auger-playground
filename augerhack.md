Certainly, to include `limitranges` and `resourcequotas` in the list of object types to be processed, you can update the `OBJECT_TYPES` array in the script. Here's the revised script with these types included:

```bash
#!/bin/bash

# Define your etcd endpoint
ETCD_ENDPOINT="localhost:2379"

# Define the list of object types you are interested in
OBJECT_TYPES=("secrets" "configmaps" "routes" "deployments" "deploymentconfigs" "services" "limitranges" "resourcequotas")

# Define the list of namespaces you want to process
NAMESPACES=("my-namespace1" "my-namespace2" "my-namespace3")

# Loop over each object type
for OBJECT_TYPE in "${OBJECT_TYPES[@]}"; do
    echo "Processing object type: $OBJECT_TYPE"

    # Loop over each namespace
    for NAMESPACE in "${NAMESPACES[@]}"; do
        echo "Processing namespace: $NAMESPACE"

        # Define the output file for the namespace
        OUTPUT_FILE="${NAMESPACE}-objects.yaml"

        # Get all keys for the object type within the specific namespace
        KEYS=$(etcdctl --endpoints=$ETCD_ENDPOINT get /openshift.io/$OBJECT_TYPE/$NAMESPACE --prefix --keys-only)
        
        # Loop over the keys to get their values
        for KEY in $KEYS; do
            # Fetch the full data for the key
            VALUE=$(etcdctl --endpoints=$ETCD_ENDPOINT get "$KEY" --print-value-only)
            
            # Decode the data using auger
            # If auger accepts value from stdin
            DECODED_VALUE=$(echo "$VALUE" | auger decode)

            # Append the decoded value to the output file, with a '---' delimiter
            {
                echo "---"
                echo "$DECODED_VALUE"
            } >> "$OUTPUT_FILE"
            
        done
    done
done

echo "All data processed and saved to respective files."
```

In this script:

- `limitranges` and `resourcequotas` have been added to the array `OBJECT_TYPES`.
- The script will process these object types along with the others previously specified.
- Make sure to adjust the `NAMESPACES` array to include the actual namespaces you want to process.

This script will create an output file for each namespace containing the YAML data for all specified object types, separated by `---`. Ensure that `auger` is installed and configured to decode etcd data to YAML format, and that you have permission to run `etcdctl` and write to the output files.
