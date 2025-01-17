# Backing up Supply Chain Security Tools – Store data

By default, the metadata store uses a `PersistentVolume` mounted on a Postgres instance, making it a stateful component of TAP. When using the provided Postgres instance, a regular backup strategy should be implemented as part of the disaster recovery plan.

[Velero](https://velero.io/) can be used as a tool to create regular backups. 

Note that backup support for `PersistentVolume` depends on the used `StorageClass` and existing provider plugins. See the officially [supported plugins here](https://velero.io/plugins/).

```bash
velero install --provider <provider> --bucket <bucket-name> --plugins <plugin-image-location> --secret-file <secrets-file>

# Example for GCP:
# velero install --provider gcp --bucket <gcs-bucket-name> --plugins velero/velero-plugin-for-gcp:v1.3.0 --secret-file <gcp-json-credentials>
```

Velero CLI can then be used to create a backup of all the resources in the `metadata-store` namespace, including `PersistentVolumeClaim` and `PersistentVolume`. 

```bash
velero backup create metadata-store-$(date '+%s') --include-namespaces=metadata-store
```

Velero CLI can then be used to restore the Store in the same or in a different  cluster. The same namespace can be used to restore, but it may collide with other installations of Supply Chain Security Tools – Store. Furthermore, restoring into the same namespace will restore a fully functional instance of Supply Chain Security Tools – Store, but this instance won't be managed by TAP and will conflict with future installations.

```bash
velero restore create restore-metadata-store-$timestamp --from-backup metadata-store-$timestamp --namespace-mappings metadata-store:metadata-store
```

Alternatively, a different namepace can be used to restore Supply Chain Security Tools – Store. In this case, Supply Chain Security Tools – Store's API won't be available due to conflicting definitions in the RBAC proxy configuration, causing all request to fail with an `Unauthorized` error. In this scenario, the postgres instance is still accessible and tools such as `pg_dump` can be used to retrieve table contents and restored in a new live installation of Supply Chain Security Tools – Store.

```bash
velero restore create restore-metadata-store-$timestamp --from-backup metadata-store-$timestamp --namespace-mappings metadata-store:restored-metadata-store
```
 
Currently, mounting a pre-existing `PersistentVolume` or `PersistentVolumeClaim` during installation is not supported.

The minimum suggested resources for backups are `PersistentVolume`, `PersistentVolumeClaim` and `Secret`. The database password `Secret` is needed to setup a Postgres instance with the correct password to be able to properly read data from the restored volume.


    





