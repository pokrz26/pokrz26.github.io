```bicep
@export()
type ImageRetentionPolicy = {
  namespacePrefix: string
  imagesToKeep: int
}

param acrName string
param imageRetentionPolicies ImageRetentionPolicy[] = []
param defaultImagesToKeep int = 10

var clearImagesPolicyCases = [for policy in imageRetentionPolicies: format(
  '        {0}*) imagesToKeep={1} ;;', 
  policy.namespacePrefix, 
  policy.imagesToKeep)
]
var clearImagesCommands = format('''
- cmd: az login --identity

- cmd: az acr login --name "{2}" --suffix "{3}" --expose-token --output tsv --query accessToken > /workspace/acr-token

- cmd: az acr repository list --name "{2}" --suffix "{3}" --output tsv > /workspace/repositories

- cmd: >-
    mcr.microsoft.com/acr/acr-cli:0.19 -c 'set -eu;
    repositoryList=/workspace/repositories;
    test -s "$repositoryList";
    token=$(cat /workspace/acr-token);
    while IFS= read -r repository; do
    imagesToKeep={0};
    case "$repository" in
    {1}
    esac;
    echo "Purging images from repository: $repository, keeping $imagesToKeep images";
    acr purge --registry "{2}" --username 00000000-0000-0000-0000-000000000000 --password "$token" --filter "$repository:.*" --ago 0d --keep "$imagesToKeep" --dry-run;
    done < "$repositoryList"'
  entryPoint: /bin/sh
  disableWorkingDirectoryOverride: true
  timeout: 3600
''', defaultImagesToKeep, join(clearImagesPolicyCases, ' '), containerRegistry.properties.loginServer, replace(replace(containerRegistry.properties.loginServer, format('{0}-', acrName), ''), '.azurecr.io', ''))

resource containerRegistry 'Microsoft.ContainerRegistry/registries@2025-11-01' existing = {
  name: acrName
}

resource clearImagesTask 'Microsoft.ContainerRegistry/registries/tasks@2025-03-01-preview' = {
  parent: containerRegistry
  name: 'Clear_images_at_monday_midnight'
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    status: 'Enabled'
    platform: {
      os: 'Linux'
      architecture: 'amd64'
    }
    timeout: 3600
    trigger: {
      timerTriggers: [
        {
          name: 'Clear images at monday midnight'
          schedule: '0 0 * * Mon'
          status: 'Enabled'
        }
      ]
    }
    step: {
      type: 'EncodedTask'
      encodedTaskContent: base64(format('version: v1.1.0\nsteps:\n{0}', clearImagesCommands))
      contextPath: '/dev/null'
    }
  }
}

resource clearImagesTaskAcrRepositoryCatalogListerRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(
    containerRegistry.id,
    clearImagesTask.identity.principalId,
    'Container Registry Repository Catalog Lister'
  )
  scope: containerRegistry
  properties: {
    roleDefinitionId: subscriptionResourceId(
      'Microsoft.Authorization/roleDefinitions',
      '<CATALOG_LISTER_ROLE_DEFINITION_ID>'
    )
    principalId: clearImagesTask.identity.principalId
    principalType: 'ServicePrincipal'
  }
}

var acrRepositoryCatalogListerRoleDefinitionId = subscriptionResourceId(
  'Microsoft.Authorization/roleDefinitions',
  'bfdb9389-c9a5-478a-bb2f-ba9ca092c3c7'
)

var acrRepositoryContributorRoleDefinitionId = subscriptionResourceId(
  'Microsoft.Authorization/roleDefinitions',
  '2efddaa5-3f1f-4df3-97df-af3f13818f4c'
)

resource clearImagesTaskAcrRepositoryCatalogListerRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(containerRegistry.id, clearImagesTask.identity.principalId, 'Container Registry Repository Catalog Lister')
  scope: containerRegistry
  properties: {
    roleDefinitionId: acrRepositoryCatalogListerRoleDefinitionId
    principalId: clearImagesTask.identity.principalId
    principalType: 'ServicePrincipal'
  }
}

resource clearImagesTaskAcrRepositoryContributorRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(containerRegistry.id, clearImagesTask.identity.principalId, 'Container Registry Repository Contributor')
  scope: containerRegistry
  properties: {
    roleDefinitionId: acrRepositoryContributorRoleDefinitionId
    principalId: clearImagesTask.identity.principalId
    principalType: 'ServicePrincipal'
  }
}
```
