### Get Bearer Token from Refresh Token:

```
export BTOKEN=$(curl \
--silent \
--data-urlencode "grant_type=refresh_token" \
--data-urlencode "client_id=cloud-services" \
--data-urlencode "refresh_token="REDHAT_OFFLINE_TOKEN"" \
https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token | \
jq -r .access_token)
```

### Cluster IDs:

#### Get Clusters:
```
curl -X GET -H "accept: application/json" -H "Authorization: Bearer $BTOKEN" "https://api.openshift.com/api/assisted-install/v2/clusters/"
```


#### Delete Assisted Installer Cluster Object
```
curl -X DELETE -H "accept: application/json" -H "Authorization: Bearer $BTOKEN" "https://api.openshift.com/api/assisted-install/v2/clusters/$CLUSTER_ID"
```


### Infra IDs:

#### Get Cluster Infra IDs:

```
curl -X GET -H "accept: application/json" -H "Authorization: Bearer $BTOKEN" "https://api.openshift.com/api/assisted-install/v2/infra-envs/"
```

#### Delete Cluster Infra ID:
```
curl -X DELETE -H "accept: application/json" -H "Authorization: Bearer $BTOKEN" "https://api.openshift.com/api/assisted-install/v2/infra-envs/$ID"
```