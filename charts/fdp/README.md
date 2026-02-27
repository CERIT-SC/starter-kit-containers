## Example values.yaml

```yaml
server:
  config:
    clientUrl: https://fdp.gdi.cloud.e-infra.cz
    persistentUrl: https://fdp.gdi.cloud.e-infra.cz
    jwtSecret: <secret>
  persistence:
    storageClassName: "nfs-csi"

mongodb:
  pvc:
    storageClass: nfs-csi

ingress:
  hosts:
    - host: 'fdp.gdi.cloud.e-infra.cz'
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
   - secretName: fdp-gdi-cloud-e-infra-cz-tls
     hosts:
       - fdp.gdi.cloud.e-infra.cz

```