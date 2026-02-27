# Loading IG into FHIR server
To load your various profiles into a FHIR server, you can use the $install 
operator. This may be useful when we are setting up the new FHIR servers
```bash
IG=https://deploy-preview-162--ncpi-fhir-ig-v2.netlify.app/package.tgz
# For netlify, we have to use --insecure, but the regular IG is fine without
# that option
curl -L --insecure -o /tmp/ig.tgz $IG

BASE64_CONTENT=$(base64 -w 0 /tmp/ig.tgz)
curl -s -X POST "http://localhost:8080/fhir/ImplementationGuide/\$install" \
  -H "Content-Type: application/json" \
  --data-binary @- <<EOF
{
  "resourceType": "Parameters",
  "parameter": [
    {
      "name": "npmContent",
      "valueBase64Binary": "$BASE64_CONTENT"
    }
  ]
}
EOF
```

A more complete script can be found [here](https://github.com/carrollaboratory/hapi-helper/blob/main/scripts/load_ig.sh)
