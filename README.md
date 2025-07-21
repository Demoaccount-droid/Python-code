#!/bin/bash

# -------- Configuration --------
JENKINS_URL="http://your-jenkins-host"
USERNAME="your-username"
API_TOKEN="your-api-token"

# List of resources to create
# Format: name|labels|description
RESOURCES=(
  "device-01|android,pixel|Pixel 6 - Automation Device"
  "device-02|android,samsung|Samsung Galaxy S22"
  "device-03|ios,ipad|iPad 10th Gen"
)

# --------------------------------

# Get CSRF crumb
CRUMB=$(curl -s -u "${USERNAME}:${API_TOKEN}" \
  "${JENKINS_URL}/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)")

if [ -z "$CRUMB" ]; then
  echo "[ERROR] Failed to get CSRF crumb."
  exit 1
fi

# Loop through and create each resource
for entry in "${RESOURCES[@]}"; do
  IFS="|" read -r NAME LABELS DESC <<< "$entry"

  echo "[INFO] Creating resource: $NAME"

  curl -s -X POST "${JENKINS_URL}/lockable-resources/createResource" \
    -H "$CRUMB" \
    --user "${USERNAME}:${API_TOKEN}" \
    --data-urlencode "name=${NAME}" \
    --data-urlencode "labels=${LABELS}" \
    --data-urlencode "description=${DESC}" \
    && echo "[SUCCESS] Created $NAME" || echo "[FAIL] Failed to create $NAME"
done
