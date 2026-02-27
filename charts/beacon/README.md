## Example values.yaml

```yaml
mongodb:
  pvc:
    storageClass: nfs-csi

beacon:
  config:
    beaconId: 'cz.e-infra.cloud.gdi.beacon-aggregated-staging'
    beaconName: 'Czech staging aggregated Beacon node'
    uri: 'https://beacon-aggregated-staging.gdi.cloud.e-infra.cz'
    welcomeUrl: 'https://beacon-aggregated-staging.gdi.cloud.e-infra.cz/'
    alternativeUrl: 'https://beacon-aggregated-staging.gdi.cloud.e-infra.cz/api'
    description: "Czech staging aggregated Beacon node"
    orgId: 'CERIT-SC'
    orgName: 'CERIT Scientific Cloud'
    orgDescription: 'A Czech national centre operating computing and data storage infrastructure for research and development.'
    orgAddress: 'Centrum CERIT-SC, Masarykova univerzita, Šumavská 416/15, 602 00 Brno, Czech Republic'
    orgWelcomeUrl: 'https://www.cerit-sc.cz/'
    orgContactUrl: 'mailto:k8s@cerit-sc.cz'
    orgLogoUrl: 'https://www.e-infra.cz/img/logo.svg'
    environment: 'test'
  datasets:
    datasetsConfig: |
      GDI-CZ-MUNI-00001:
        isSynthetic: true
        isTest: true
    datasetsPermissions: |
      GDI-CZ-MUNI-00001:
        public:
          default_entry_types_granularity: record
          entry_types_exceptions:
            - individual: record

riTools:
  enabled: true
  pvc:
    size: 10Gi
    storageClass: "nfs-csi"
  resources:
    limits:
      cpu: 2
      memory: 4Gi
    requests:
      cpu: 1
      memory: 2Gi
ingress:
  enabled: true
  className: nginx
  annotations:
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  host: beacon-aggregated-staging.gdi.cloud.trusted.e-infra.cz
```