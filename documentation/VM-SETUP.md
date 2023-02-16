
# FarmVibes.AI VM Setup

This documents assists you on creating a new Azure VM that is capable of running
FarmVibes.AI from scratch.

## Requirements

* You'll need a Linux computer or Windows Subsystem for Linux in your Windows Machine.
* The Azure CLI will need to be installed. More info [here](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).
* You'll need an Azure Subscription. More details on how to setup one can be found [here](https://azure.microsoft.com/en-us/free/search/?ef_id=EAIaIQobChMIlcaGu9bG-gIVET6RCh3KXwK9EAAYASAAEgLP-vD_BwE%3AG%3As&OCID=AIDcmmzmnb0182_SEM_EAIaIQobChMIlcaGu9bG-gIVET6RCh3KXwK9EAAYASAAEgLP-vD_BwE%3AG%3As&gclid=EAIaIQobChMIlcaGu9bG-gIVET6RCh3KXwK9EAAYASAAEgLP-vD_BwE).
* Once you have access to a subscription, you'll need to create a resource group or use an existing
resource group for which you have at least a `contributor` role.
* You'll also need a ssh public key (ideally in the default location at `~/.ssh/id_rsa.pub`). If you don't have
a ssh key, one can be generated by running `ssh-keygen` in your shell. When creating the VM, we add this public key  to your VM so you can log in easily (more info on SSH keys for accessing VMs are available [here](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys)).
* You'll need access to two files in the FarmVibes repo, `farmvibes_ai_vm.bicep` and `setup_farmvibes_ai_vm.sh` both in the folder `resources/vm/`. To access these files you can copy them directly from our [repo](https://github.com/microsoft/farmvibes-ai) or clone the whole repo to your local computer.

## Creating a new Ubuntu VM

1. Login into Azure using Azure CLI:
```shell
az login
```

2. If you have multiple Azure Subscriptions, select the subscription that you want to use:
```shell
az account set --subscription <SUBSCRIPTION NAME>
```

3. From the repo root, you'll need to run the following deployment command:
```shell
az deployment group create --resource-group <resource_group> \
   --name <deployment_name> \
   --template-file  resources/vm/farmvibes_ai_vm.bicep \
   --parameters \
            ssh_public_key="$(cat ~/.ssh/id_rsa.pub)" \
            vm_suffix_name=<my_test_suffix> \
            encoded_script="$(cat resources/vm/setup_farmvibes_ai_vm.sh | gzip -9 | base64 -w0)"
```
Please, change `<resource_group>`, `<deployment_name>`, and `<my_test_suffix>`
to names of your preference.

* `<resource_group>` refers to the resource group the VM to be deployed.

* `<deployment_name>` specifies the name of this VM deployment. If you do not
  pass this argument, `az cli` assumes the deployment name as the bicep file
  file.

* `<my_test_suffix>`. VMs are created with the prefix `farmvibes-ai-vm-`. Then,
  if you create a VM with suffix `testvibes`, the machine name should be
  `farmvibes-ai-vm-testvibes`. Azure VM names cannot can't use spaces, control
  characters, or these
  [characters](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules)
  `~ ! @ # $ % ^ & * ( ) = + _ [ ] { } \ | ; : . ' " , < > / ?`.

You can see the list of VM parameters in the file `resources/vm/farmvibes_ai_vm.bicep`.

4. Once the script completes,  a JSON describing the resources created will be printed in the shell. You can get the ssh connection command with the following command.

```
az deployment group show \
  -g <resource_group> \
  -n <deployment_name> \
  --query properties.outputs.ssh_command.value
```

Once the VM is succefully created, you can follow the steps on the [quickstart guide](../QUICKSTART.md) to install FarmVibes.AI
and get it operational. Please note that all the required dependencies (such as docker) will already be installed in the VM.