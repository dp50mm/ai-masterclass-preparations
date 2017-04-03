# GCP create GPU instance

Compiled guide on how to setup GPU instances.
Make sure to request additional GPUs for region europe-west1:
https://cloud.google.com/compute/quotas#requesting_additional_quota

This guide requires the 'gcloud' commandline utils to be installed:
https://cloud.google.com/sdk/docs/quickstart-mac-os-x
https://cloud.google.com/sdk/docs/quickstart-windows

The following commands are to configure gcloud.
Setting a default zone makes it easier to enter the instance creation commands but is not a requirement.


Create configuration profile 'default':

- ```gcloud config configurations activate default```

Set zone to 'europe-west-1b':

- ```gcloud config set compute/zone europe-west1-b```

Set region to 'europe':

- ```gcloud config set compute/region europe-west1```

Check gcloud version to be at least 144.0.0:
- ```gcloud version```

Update, and install gcloud beta components:
- ```gcloud components update```
- ```gcloud components install beta```

Beta components are required because GPUs for Compute Engine are still in beta.

Make sure to set gcloud to the project you want to create instances with:
- ```gcloud config set project [YOUR PROJECT-ID]```

Check if you have enough GPUs available:
- ```gcloud beta compute regions describe 'europe-west1'```

# Creating GPU instance and installing Nvidia drivers.

Compute Engine uses Nvidia K80 GPUs and those require nvidia drivers to be installed, these do not come by default in the linux images.

The startup script in the example can also be used as guide on how to install Nvidia drivers manually when connected with SSH.

Create instance with GPUs:
```sh

gcloud beta compute instances create gpu-test-1 \
    --machine-type n1-standard-2 --zone europe-west1-b \
    --accelerator type=nvidia-tesla-k80,count=1 \
    --image-family ubuntu-1604-lts --image-project ubuntu-os-cloud \
    --maintenance-policy TERMINATE --restart-on-failure \
    --metadata startup-script='#!/bin/bash
    echo "Checking for CUDA and installing."
    # Check for CUDA and try to install.
    if ! dpkg-query -W cuda; then
      curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
      dpkg -i ./cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
      apt-get update
      apt-get install cuda -y
    fi'
```

Parameter values:
```sh

gcloud beta compute instances create [INSTANCE_NAME] \
    --machine-type [MACHINE_TYPE] --zone [ZONE] \
    --accelerator type=[ACCELERATOR_TYPE],count=[ACCELERATOR_COUNT] \
    --image-family [IMAGE_FAMILY] --image-project [IMAGE_PROJECT] \
    --maintenance-policy TERMINATE --restart-on-failure \
    --metadata startup-script='[STARTUP_SCRIPT]'

```

Verify Nvidia drivers install: ```nvidia-smi```

Check simply creating an instance first, to see if gcloud and your project works at all:
```

gcloud beta compute instances create gpu-test-1 \
    --machine-type n1-standard-2 --zone europe-west1-b \
    --image-family ubuntu-1604-lts --image-project ubuntu-os-cloud \
    --maintenance-policy TERMINATE --restart-on-failure

```

## Instance commands:

Terminate an instance with:
- ```gcloud compute instances stop [INSTANCE-NAME]```

List instances:
- ```gcloud compute instances list```


Example:
```
Igors-MacBook-Pro:ai-masterclass-preparations igor$ sudo gcloud compute instances list
NAME        ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
gpu-test-1  europe-west1-b  n1-standard-2               1.1.1.1   1.1.1.1  RUNNING
```
