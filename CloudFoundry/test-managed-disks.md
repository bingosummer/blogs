>NOTE: The known regions which support managed disks are eastasia and southeastasia.

## A new deployment with a default storage account

1. Deploy the bosh-setup template.

  * The location should be Southeast Asia.

  * Disable the parameter "autoDeployBosh"

1. Update bosh.yml, and deploy BOSH.

  1. chmod +w bosh.yml

  1. Update the CPI release.

    1. Click https://binximdstorageaccount.blob.core.windows.net/cpi-release?comp=list

    1. Download the latest version (`20+dev.109.tgz`) of CPI release into the devbox.

      ```
      wget https://binximdstorageaccount.blob.core.windows.net/cpi-release/bosh-azure-cpi-20+dev.109.tgz
      ```

    1. Update bosh.yml

      ```
      - name: bosh-azure-cpi
        url: file://~/bosh-azure-cpi-20+dev.109.tgz
      ```

    1. Remove `storage_account_name`

    1. Add `use_managed_disks`

      ```
      azure: &azure
        environment: AzureCloud
        ...
        default_security_group: nsg-bosh
        use_managed_disks: true
      ```

    1. Run `./deploy_bosh.sh`

1. Deploy CF.

  Run `./deploy_cloudfoundry.sh <manifest>`


## Upgrading from an existing blob based deployment

1. You already have an existing blob based CF deployment in Southeast Asia.

1. Update bosh.yml, and re-deploy BOSH.

  1. chmod +w bosh.yml

  1. Update the CPI release.

    1. Click https://binximdstorageaccount.blob.core.windows.net/cpi-release?comp=list

    1. Download the latest version (`20+dev.109.tgz`) of CPI release into the devbox.

      ```
      wget https://binximdstorageaccount.blob.core.windows.net/cpi-release/bosh-azure-cpi-20+dev.109.tgz
      ```

    1. Update bosh.yml

      ```
      - name: bosh-azure-cpi
        url: file://~/bosh-azure-cpi-20+dev.109.tgz
      ```

    1. Add `use_managed_disks`

      ```
      azure: &azure
        environment: AzureCloud
        ...
        default_security_group: nsg-bosh
        use_managed_disks: true
      ```

    1. Run `./deploy_bosh.sh`

1. Perform a full migration.

  ```
  bosh recreate --force
  ```

1. Setup the env variables for the default storage account. Check the tagged blob based disks. You can get the metadata.

  ```
  "metadata": {
    "user_agent": "bosh",
    "migrated": "true"
  }
  ```

  ```
  export AZURE_STORAGE_ACCOUNT="<name>"
  export AZURE_STORAGE_ACCESS_KEY="<key>"
  azure storage blob list --container bosh -p bosh-data --json
  ```

## A new deployment without a default storage account

1. Deploy a simple environment in Southeast Asia.

  https://github.com/bingosummer/azure-quickstart-templates/tree/simple-bosh-setup/simple-bosh-setup
  
  In this deployment, there are no storage accounts and devbox. Only the neccessary network resources are created.
  The following steps are done in the devbox of the previous deployment "A new deployment with a default storage account".

1. Copy bosh.yml to bosh.no-default-storage-account.yml

1. Update bosh.no-default-storage-account.yml, and deploy BOSH.

  1. Remove `storage_account_name` in bosh.yml.
  
  1. Update `resource_group_name` to the new resource group name.
  
  1. Add the public IP for the BOSH VM.
  
    ```
    networks:
    - {name: private, static_ips: [10.0.0.4], default: [dns, gateway]}
    - {name: public, static_ips: [52.163.225.115]}
    
    ssh_tunnel:
    host: 52.163.225.115
    port: 22
    user: vcap # The user must be as same as above ssh_user
    private_key: ~/bosh
    
    mbus: https://mbus-user:4rndLXEyPWSzODMufg@52.163.225.115:6868
    ```
  
  1. Deploy BOSH

    ```
    export BOSH_INIT_LOG_LEVEL="Debug"
    export BOSH_INIT_LOG_PATH="./run.no-default-storage.log"
    bosh-init deploy bosh.no-default-storage-account.yml
    ```

1. Deploy CF (Optional)

  Update the director uuid and public IP of manifest.

  Run `./deploy_cloudfoundry.sh <manifest>`


## A new deployment without a default storage account (cross-region)

1. Deploy a simple environment in Southeast Asia.

  https://github.com/bingosummer/azure-quickstart-templates/tree/simple-bosh-setup/simple-bosh-setup
  
  In this deployment, there are no storage accounts and devbox. Only the neccessary network resources are created.
  The following steps are done in the devbox of the previous deployment "A new deployment with a default storage account".

1. Create a resource group in East Asia, and move all the resources into this group.

1. Copy bosh.yml to bosh.cross-region.yml

1. Update bosh.cross-region.yml, and deploy BOSH.

  1. Remove `storage_account_name` in bosh.yml.
  
  1. Update `resource_group_name` to the new resource group name.
  
  1. Add the public IP （postfix: "-bosh"） for the BOSH VM.
  
    ```
    networks:
    - {name: private, static_ips: [10.0.0.4], default: [dns, gateway]}
    - {name: public, static_ips: [52.163.225.115]}
    
    ssh_tunnel:
    host: 52.163.225.115
    port: 22
    user: vcap # The user must be as same as above ssh_user
    private_key: ~/bosh
    
    mbus: https://mbus-user:4rndLXEyPWSzODMufg@52.163.225.115:6868
    ```

  1. Add `storage_account_locaiton: southeastasia` in `resource_pool`.

  1. Deploy BOSH
  
    ```
    export BOSH_INIT_LOG_LEVEL="Debug"
    export BOSH_INIT_LOG_PATH="./run.cross-region.log"
    bosh-init deploy bosh.cross-region.yml
    ```

1. Deploy CF (Optional)

  Update the director uuid and public IP of manifest.

  Add `storage_account_location` in `cloud_properties` of each `resource_pool` and `compilation`.

  ```
  compilation:
    cloud_properties:
      instance_type: Standard_F4
      storage_account_location: southeastasia
    network: cf1
    reuse_compilation_vms: true
    workers: 6
  
  Each resource pool should be updated.
    - cloud_properties:
      instance_type: Standard_F2
      storage_account_location: southeastasia
    name: colocated_z3
    network: diego3
    stemcell:
      name: bosh-azure-hyperv-ubuntu-trusty-go_agent
      version: latest
  ```

  Run `./deploy_cloudfoundry.sh <manifest>`
