# Correlation Rules Service API

This document describes Correlation Service API to store and retrieve Correlation and Detection rules templates.

## Asset Repository Service
The Rule Service is in fact so-called Asset Repository Service (AR). AR Service was designed as a general purpose store of 
entities called assets, organize those assets to packages and version them.

Each asset has:
* `type` - to distinguish different asset type
* `name` - unique name per workspace and package
* `metadata` - key-value structure
* `data` - content of the asset

There are three more interesting properties on asset which drives asset visibility:
* `isSystem` - the asset is build-in, e.g. visible for everybody, such assets can be created and modified by system admin only.
* `isAccShared` - the asset is visible to all account users.
* `isEnterprise` - This flag means that this asset can only be seen by users who have a higher tariff than the free tariff. Only system admin user can create|modify such asset
* `dataAccessible` - Whether the user can see the correlation and detection rule stored in the `data` attribute. Computed attribute based on current user billing plan.

## Correlation (CR) and Detection Rules (DR) Assets

For Correlation Rules we introduced two new asset `type`s:
* `detection-rule` - to store Detection Rules Templates
* `correlation-rule` - for Correlation Rules Templates

CR and DR Asset `name` needs to be unique. Let's set following rule for both:


```javascript
name = sanitizeName(`${applicationName} ${applicationVersion}`)

function sanitizeName(name) {
    return name.replace(/[^a-zA-Z0-9]/g,'_')
}
```

CR and DR `metadata` must at least contain:

```json

{
  "metadata": {
    "application" : "${applicationName}",
    "version": "${applicationVersion}"
  }
}

```

CR and DR data contain JSON with template definition.

So for example the CR asset can look like:
```json
{
    "name": "wordpress_1_1_2",
    "type": "correlation-rule",
    "metadata": {
      "application": "Wordpress",
      "version": "1.1.2"
    },
    "isSystem": false,
    "isEnterprise": false,
    "isAccShared": false
}
```
and corresponding detection rule:

```json
{
    "name": "siebel_8_1_2",
    "type": "detection-rule",
    "metadata": {
      "application": "Siebel",
      "version": "8.1.2"
    },
    "isSystem": false,
    "isEnterprise": false,
    "isAccShared": false
}
```

## Use-cases

For all API calls client has to pass Basic Authentication header, where user name is API key ID and password is user API key secret.

The `${arUrl}` variable in the examples is Asset Repository URL. In case of production it is `https://ar.blazemeter.com`.
The `${wsId}` is id of users workspace.

### List all Application dnd versions

```
GET ${arUrl}/api/v1/workspaces/${wsId}/assets?withSystem=true&withAccShared=true&limit=100&skip=0&q=type=correlation-rule

{
    "limit": 100,
    "skip": 0,
    "total": 3,
    "timestamp": "2023-02-28T16:26:55+00:00",
    "request_id": "7197b196a51fdcff395b2e3465c119aa",
    "result": [
        {
            "metadata": {
                "version": "8.1.2",
                "application": "Siebel"
            },
            "id": "12c17f50-1e1d-42ff-9d4a-e8b6a8edd7c0",
            "accountId": 514413,
            "workspaceId": 1152633,
            "packageId": "d8fb2748-3474-4c8e-bf73-77030ad691b0",
            "name": "siebel_8_1_2",
            "displayName": "siebel_8_1_2",
            "type": "correlation-rule",
            "createdAt": "2023-02-28T14:31:34.561Z",
            "createdBy": 1620964,
            "updatedAt": "2023-02-28T14:31:34.561Z",
            "updatedBy": 1620964,
            "isSystem": false,
            "isEnterprise": false,
            "isAccShared": false,
            "dataAccessible": true,
            "createdByEmail": "okepka@perforce.com",
            "updatedByEmail": "okepka@perforce.com"
        },
        {
            "metadata": {
                "version": "8.1.1",
                "application": "Siebel"
            },
            "id": "4bf07ba8-2179-4f98-b6bc-2b6d6650acee",
            ...
        },
        {
            "metadata": {
                "version": "1.1.2",
                "application": "Wordpress"
            },
            "id": "f32cdbca-32a6-4722-9a66-5cd54f4f09e8",
            ...
        }
    ]
}
```

All the applications and version can be read from `metadata`.


### Get all CR versions for an Application name

For example for Siebel application name:
```
GET ${arUrl}/api/v1/workspaces/${wsId}/assets?withSystem=true&withAccShared=true&limit=100&skip=0&q=type=correlation-rule&q=metadata.application=Siebel
```

### Get CR asset for Application name and version

```
GET ${arUrl}/api/v1/workspaces/${wsId}/assets?withSystem=true&withAccShared=true&limit=100&skip=0&q=type=correlation-rule&q=metadata.application=Siebel&metadata.version=8.1.1
```

### Get CR template for and CR asset

`${crAssetId}` is asset from one of the previous calls, for example `12c17f50-1e1d-42ff-9d4a-e8b6a8edd7c0` 
```
GET ${arUrl}/api/v1/workspaces/${wsId}/assets/${crAssetId}/data
```

### Add CR

First we need to add asset.

```
POST ${arUrl}/api/v1/workspaces/${wsId}/assets
body: 
{
    "name": "wordpress_1_1_2",
    "type": "correlation-rule",
    "metadata": {
      "application": "Wordpress",
      "version": "1.1.2"
    },
    "isSystem": false,
    "isEnterprise": false,
    "isAccShared": false
}

Response:
{
    "timestamp": "2023-02-28T14:52:27+00:00",
    "request_id": "13bcc1bcc9ab4f09fd6db83bf15a1826",
    "result": {
        "id": "f32cdbca-32a6-4722-9a66-5cd54f4f09e8",
        "accountId": 514413,
        "workspaceId": 1152633,
        "packageId": "d8fb2748-3474-4c8e-bf73-77030ad691b0",
        "name": "wordpress_1_1_2",
        "displayName": "wordpress_1_1_2",
        "type": "correlation-rule",
        "metadata": {
            "application": "Wordpress",
            "version": "1.1.2"
        },
        "createdAt": "2023-02-28T14:52:27.548Z",
        "createdBy": 1620964,
        "updatedAt": "2023-02-28T14:52:27.548Z",
        "updatedBy": 1620964,
        "isSystem": false,
        "isEnterprise": false,
        "isAccShared": false
    }
}
```

Then we use asset id to add data:

```
POST ${arUrl}/api/v1/workspaces/1152633/assets/f32cdbca-32a6-4722-9a66-5cd54f4f09e8/data
body:

{
  the template
}
```

Similarly, for DR.







