```bash

#!/bin/bash

# Define your etcd endpoint
ETCD_ENDPOINT="localhost:2379"

# Define the list of Kubernetes object types you are interested in
# 'routes' and 'deploymentconfigs' have been removed as they are specific to OpenShift and not handled by auger
OBJECT_TYPES=("secrets" "configmaps" "deployments" "services" "limitranges" "resourcequotas")

# Define the exact namespaces you want to process
NAMESPACES=("tim-dev") # Add other namespaces to this array if needed

# Initialize counter for backups
COUNTER=0

# Loop over each object type
for OBJECT_TYPE in "${OBJECT_TYPES[@]}"; do
    echo "Processing object type: $OBJECT_TYPE"

    # Loop over each namespace
    for NAMESPACE in "${NAMESPACES[@]}"; do
        echo "Processing namespace: $NAMESPACE"

        # Define the output file for the namespace
        OUTPUT_FILE="${NAMESPACE}-objects.yaml"

        # Check if the output file exists
        if [ -f "$OUTPUT_FILE" ]; then
            # Create a backup with the current date-time in minutes and a counter
            BACKUP_TIME=$(date +"%Y%m%d%H%M")
            BACKUP_FILE="${OUTPUT_FILE}-${BACKUP_TIME}-${COUNTER}"
            mv "$OUTPUT_FILE" "$BACKUP_FILE"
            echo "Backup created: $BACKUP_FILE"
            ((COUNTER++))
        fi

        # Create a new output file
        touch "$OUTPUT_FILE"

        # Get all keys for the object type within the specific namespace
        KEYS=$(etcdctl --endpoints=$ETCD_ENDPOINT get /kubernetes.io/$OBJECT_TYPE/$NAMESPACE --prefix --keys-only | grep -E "/$NAMESPACE(/|$)")

        # Process the keys if any are found
        if [ -n "$KEYS" ]; then
            for KEY in $KEYS; do

                # Fetch the full data for the key
                VALUE=$(etcdctl --endpoints=$ETCD_ENDPOINT get "$KEY" --print-value-only)
                
                # Decode the data using auger
                DECODED_VALUE=$(echo "$VALUE" | auger decode)

                # Append the decoded value to the output file, with '---' as delimiter
                {
                    echo "---"
                    echo "$DECODED_VALUE"
                } >> "$OUTPUT_FILE"
                
            done
        else
            echo "No keys found for namespace: $NAMESPACE and object type: $OBJECT_TYPE"
        fi
    done
done

echo "All data processed and saved to respective files."

```
