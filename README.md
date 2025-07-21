#!/bin/bash

# ------------- CONFIG -------------
JENKINS_URL="http://your-jenkins-url"        # Example: http://jenkins.local:8080
USERNAME="your-username"
API_TOKEN="your-api-token"

# Define resources in "name|labels|description" format
RESOURCES=(
  "device-001|android,usb|Pixel 6 Android USB device"
  "device-002|ios,lightning|iPhone 13 Lightning device"
  "device-003|android,wifi|Samsung Galaxy Tab - WiFi"
)
# ----------------------------------

# Get CSRF crumb for Jenkins API
CRUMB=$(curl -s -u "${USERNAME}:${API_TOKEN}" \
  "${JENKINS_URL}/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)")

if [ -z "$CRUMB" ]; then
  echo "[ERROR] Failed to fetch CSRF crumb. Check Jenkins URL and credentials."
  exit 1
fi

# Add each resource
for entry in "${RESOURCES[@]}"; do
  IFS="|" read -r NAME LABELS DESCRIPTION <<< "$entry"

  echo "[INFO] Creating resource: ${NAME}"

  curl -s -X POST "${JENKINS_URL}/lockable-resources/createResource" \
    -H "$CRUMB" \
    --user "${USERNAME}:${API_TOKEN}" \
    --data-urlencode "name=${NAME}" \
    --data-urlencode "labels=${LABELS}" \
    --data-urlencode "description=${DESCRIPTION}"

  echo "[SUCCESS] Resource '${NAME}' created with labels '${LABELS}' and description '${DESCRIPTION}'."
done
