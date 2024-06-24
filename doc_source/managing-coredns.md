# Working with the CoreDNS Amazon EKS add\-on<a name="managing-coredns"></a>

CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS\. When you launch an Amazon EKS cluster with at least one node, two replicas of the CoreDNS image are deployed by default, regardless of the number of nodes deployed in your cluster\. The CoreDNS Pods provide name resolution for all Pods in the cluster\. The CoreDNS Pods can be deployed to Fargate nodes if your cluster includes an [AWS Fargate profile](fargate-profile.md) with a namespace that matches the namespace for the CoreDNS `deployment`\. For more information about CoreDNS, see [Using CoreDNS for Service Discovery](https://kubernetes.io/docs/tasks/administer-cluster/coredns/) in the Kubernetes documentation\.

The following table lists the latest version of the Amazon EKS add\-on type for each Kubernetes version\.<a name="coredns-versions"></a>


| Kubernetes version | `1.27` | `1.26` | `1.25` | `1.24` | `1.23` | 
| --- | --- | --- | --- | --- | --- | 
|  | v1\.10\.1\-eksbuild\.3 | v1\.9\.3\-eksbuild\.6 | v1\.9\.3\-eksbuild\.6 | v1\.9\.3\-eksbuild\.6 | v1\.8\.7\-eksbuild\.7 | 

**Important**  
If you're self\-managing this add\-on, the versions in the table might not be the same as the available self\-managed versions\. For more information about updating the self\-managed type of this add\-on, see [Updating the self\-managed add\-on](#coredns-add-on-self-managed-update)\.
To improve the stability and availability of the CoreDNS Deployment, versions `v1.9.3-eksbuild.5` and later and `v1.10.1-eksbuild.2` are deployed with a `PodDisruptionBudget`\. If you've deployed an existing `PodDisruptionBudget`, your upgrade to these versions might fail\. If the upgrade fails, completing one of the following tasks should resolve the issue:  
When doing the upgrade of the Amazon EKS add\-on, choose to override the existing settings as your conflict resolution option\. If you've made other custom settings to the Deployment, make sure to back up your settings before upgrading so that you can reapply your other custom settings after the upgrade\.
Remove your existing `PodDisruptionBudget` and try the upgrade again\.

**Prerequisites**
+ An existing Amazon EKS cluster\. To deploy one, see [Getting started with Amazon EKS](getting-started.md)\.

## Creating the Amazon EKS add\-on<a name="coredns-add-on-create"></a>

Create the Amazon EKS type of the add\-on\.

1. See which version of the add\-on is installed on your cluster\.

   ```
   kubectl describe deployment coredns --namespace kube-system | grep coredns: | cut -d : -f 3
   ```

   An example output is as follows\.

   ```
   v1.9.3-eksbuild.6
   ```

1. See which type of the add\-on is installed on your cluster\. Depending on the tool that you created your cluster with, you might not currently have the Amazon EKS add\-on type installed on your cluster\. Replace *my\-cluster* with the name of your cluster\.

   ```
   aws eks describe-addon --cluster-name my-cluster --addon-name coredns --query addon.addonVersion --output text
   ```

   If a version number is returned, you have the Amazon EKS type of the add\-on installed on your cluster and don't need to complete the remaining steps in this procedure\. If an error is returned, you don't have the Amazon EKS type of the add\-on installed on your cluster\. Complete the remaining steps of this procedure to install it\.

1. Save the configuration of your currently installed add\-on\.

   ```
   kubectl get deployment coredns -n kube-system -o yaml > aws-k8s-coredns-old.yaml
   ```

1. Create the add\-on using the AWS CLI\. If you want to use the AWS Management Console or `eksctl` to create the add\-on, see [Creating an add\-on](managing-add-ons.md#creating-an-add-on) and specify `coredns` for the add\-on name\. Copy the command that follows to your device\. Make the following modifications to the command, as needed, and then run the modified command\.
   + Replace `my-cluster` with the name of your cluster\.
   + Replace *v1\.10\.1\-eksbuild\.3* with the latest version listed in the [latest version table](#coredns-versions) for your cluster version\.

   ```
   aws eks create-addon --cluster-name my-cluster --addon-name coredns --addon-version v1.10.1-eksbuild.3
   ```

   If you've applied custom settings to your current add\-on that conflict with the default settings of the Amazon EKS add\-on, creation might fail\. If creation fails, you receive an error that can help you resolve the issue\. Alternatively, you can add **\-\-resolve\-conflicts OVERWRITE** to the previous command\. This allows the add\-on to overwrite any existing custom settings\. Once you've created the add\-on, you can update it with your custom settings\.

1. Confirm that the latest version of the add\-on for your cluster's Kubernetes version was added to your cluster\. Replace `my-cluster` with the name of your cluster\.

   ```
   aws eks describe-addon --cluster-name my-cluster --addon-name coredns --query addon.addonVersion --output text
   ```

   It might take several seconds for add\-on creation to complete\.

   An example output is as follows\.

   ```
   v1.10.1-eksbuild.3
   ```

1. If you made custom settings to your original add\-on, before you created the Amazon EKS add\-on, use the configuration that you saved in a previous step to [update](#coredns-add-on-update) the Amazon EKS add\-on with your custom settings\.

## Updating the Amazon EKS add\-on<a name="coredns-add-on-update"></a>

Update the Amazon EKS type of the add\-on\. If you haven't added the Amazon EKS type of the add\-on to your cluster, either [add it](#coredns-add-on-create) or see [Updating the self\-managed add\-on](#coredns-add-on-self-managed-update), instead of completing this procedure\.

1. See which version of the add\-on is installed on your cluster\. Replace `my-cluster` with your cluster name\.

   ```
   aws eks describe-addon --cluster-name my-cluster --addon-name coredns --query "addon.addonVersion" --output text
   ```

   An example output is as follows\.

   ```
   v1.9.3-eksbuild.6
   ```

   If the version returned is the same as the version for your cluster's Kubernetes version in the [latest version table](#coredns-versions), then you already have the latest version installed on your cluster and don't need to complete the rest of this procedure\. If you receive an error, instead of a version number in your output, then you don't have the Amazon EKS type of the add\-on installed on your cluster\. You need to [create the add\-on](#coredns-add-on-create) before you can update it with this procedure\.

1. Save the configuration of your currently installed add\-on\.

   ```
   kubectl get deployment coredns -n kube-system -o yaml > aws-k8s-coredns-old.yaml
   ```

1. Update your add\-on using the AWS CLI\. If you want to use the AWS Management Console or `eksctl` to update the add\-on, see [Updating an add\-on](managing-add-ons.md#updating-an-add-on)\. Copy the command that follows to your device\. Make the following modifications to the command, as needed, and then run the modified command\.
   + Replace `my-cluster` with the name of your cluster\.
   + Replace *v1\.10\.1\-eksbuild\.3* with the latest version listed in the [latest version table](#coredns-versions) for your cluster version\.
   + The **\-\-resolve\-conflicts** *PRESERVE* option preserves existing configuration values for the add\-on\. If you've set custom values for add\-on settings, and you don't use this option, Amazon EKS overwrites your values with its default values\. If you use this option, then we recommend testing any field and value changes on a non\-production cluster before updating the add\-on on your production cluster\. If you change this value to `OVERWRITE`, all settings are changed to Amazon EKS default values\. If you've set custom values for any settings, they might be overwritten with Amazon EKS default values\. If you change this value to `none`, Amazon EKS doesn't change the value of any settings, but the update might fail\. If the update fails, you receive an error message to help you resolve the conflict\. 
   + If you're not updating a configuration setting, remove **\-\-configuration\-values '\{*"replicaCount":3*\}'** from the command\. If you're updating a configuration setting, replace *"replicaCount":3* with the setting that you want to set\. In this example, the number of replicas of CoreDNS is set to `3`\. The value that you specify must be valid for the configuration schema\. If you don't know the configuration schema, run **aws eks describe\-addon\-configuration \-\-addon\-name coredns \-\-addon\-version *v1\.10\.1\-eksbuild\.3***, replacing *v1\.10\.1\-eksbuild\.3* with the version number of the add\-on that you want to see the configuration for\. The schema is returned in the output\. If you have any existing custom configuration, want to remove it all, and set the values for all settings back to Amazon EKS defaults, remove *"replicaCount":3* from the command, so that you have empty `{}`\. For more information about CoreDNS settings, see [Customizing DNS Service](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/) in the Kubernetes documentation\.

     ```
     aws eks update-addon --cluster-name my-cluster --addon-name coredns --addon-version v1.10.1-eksbuild.3 \
         --resolve-conflicts PRESERVE --configuration-values '{"replicaCount":3}'
     ```

     It might take several seconds for the update to complete\.

1. Confirm that the add\-on version was updated\. Replace `my-cluster` with the name of your cluster\.

   ```
   aws eks describe-addon --cluster-name my-cluster --addon-name coredns
   ```

   It might take several seconds for the update to complete\.

   An example output is as follows\.

   ```
   {
       "addon": {
           "addonName": "coredns",
           "clusterName": "my-cluster",
           "status": "ACTIVE",
           "addonVersion": "v1.10.1-eksbuild.3",
           "health": {
               "issues": []
           },
           "addonArn": "arn:aws:eks:region:111122223333:addon/my-cluster/coredns/d2c34f06-1111-2222-1eb0-24f64ce37fa4",
           "createdAt": "2023-03-01T16:41:32.442000+00:00",
           "modifiedAt": "2023-03-01T18:16:54.332000+00:00",
           "tags": {},
           "configurationValues": "{\"replicaCount\":3}"
       }
   }
   ```

## Updating the self\-managed add\-on<a name="coredns-add-on-self-managed-update"></a>

**Important**  
We recommend adding the Amazon EKS type of the add\-on to your cluster instead of using the self\-managed type of the add\-on\. If you're not familiar with the difference between the types, see [Amazon EKS add\-ons](eks-add-ons.md)\. For more information about adding an Amazon EKS add\-on to your cluster, see [Creating an add\-on](managing-add-ons.md#creating-an-add-on)\. If you're unable to use the Amazon EKS add\-on, we encourage you to submit an issue about why you can't to the [Containers roadmap GitHub repository](https://github.com/aws/containers-roadmap/issues)\.

1. Confirm that you have the self\-managed type of the add\-on installed on your cluster\. Replace *my\-cluster* with the name of your cluster\.

   ```
   aws eks describe-addon --cluster-name my-cluster --addon-name coredns --query addon.addonVersion --output text
   ```

   If an error message is returned, you have the self\-managed type of the add\-on installed on your cluster\. Complete the remaining steps in this procedure\. If a version number is returned, you have the Amazon EKS type of the add\-on installed on your cluster\. To update the Amazon EKS type of the add\-on, use the procedure in [Updating the Amazon EKS add\-on](#coredns-add-on-update), rather than using this procedure\. If you're not familiar with the differences between the add\-on types, see [Amazon EKS add\-ons](eks-add-ons.md)\.

1. See which version of the container image is currently installed on your cluster\.

   ```
   kubectl describe deployment coredns -n kube-system | grep Image | cut -d ":" -f 3
   ```

   An example output is as follows\.

   ```
   v1.8.7-eksbuild.2
   ```

1. If your current CoreDNS version is `v1.5.0` or later, but earlier than the version listed in the [CoreDNS versions](#coredns-versions) table, then skip this step\. If your current version is earlier than `1.5.0`, then you need to modify the `ConfigMap` for CoreDNS to use the forward add\-on, rather than the proxy add\-on\.

   1. Open the configmap with the following command\.

      ```
      kubectl edit configmap coredns -n kube-system
      ```

   1. Replace `proxy` in the following line with `forward`\. Save the file and exit the editor\.

      ```
      proxy . /etc/resolv.conf
      ```

1. If you originally deployed your cluster on Kubernetes `1.17` or earlier, then you may need to remove a discontinued line from your CoreDNS manifest\.
**Important**  
You must complete this step before updating to CoreDNS version `1.7.0`, but it's recommended that you complete this step even if you're updating to an earlier version\. 

   1. Check to see if your CoreDNS manifest has the line\.

      ```
      kubectl get configmap coredns -n kube-system -o jsonpath='{$.data.Corefile}' | grep upstream
      ```

      If no output is returned, your manifest doesn't have the line and you can skip to the next step to update CoreDNS\. If output is returned, then you need to remove the line\.

   1. Edit the `ConfigMap` with the following command, removing the line in the file that has the word `upstream` in it\. Do not change anything else in the file\. Once the line is removed, save the changes\.

      ```
      kubectl edit configmap coredns -n kube-system -o yaml
      ```

1. Retrieve your current CoreDNS image version:

   ```
   kubectl describe deployment coredns -n kube-system | grep Image
   ```

   An example output is as follows\.

   ```
   602401143452.dkr.ecr.region-code.amazonaws.com/eks/coredns:v1.8.7-eksbuild.2
   ```

1. If you're updating to CoreDNS `1.8.3` or later, then you need to add the `endpointslices` permission to the `system:coredns` Kubernetes `clusterrole`\.

   ```
   kubectl edit clusterrole system:coredns -n kube-system
   ```

   Add the following lines under the existing permissions lines in the `rules` section of the file\.

   ```
   [...]
   - apiGroups:
     - discovery.k8s.io
     resources:
     - endpointslices
     verbs:
     - list
     - watch
   [...]
   ```

1. Update the CoreDNS add\-on by replacing `602401143452` and `region-code` with the values from the output returned in a previous step\. Replace *`v1.10.1-eksbuild.3`* with the CoreDNS version listed in the [latest versions table](#coredns-versions) for your Kubernetes version\.

   ```
   kubectl set image deployment.apps/coredns -n kube-system  coredns=602401143452.dkr.ecr.region-code.amazonaws.com/eks/coredns:v1.10.1-eksbuild.3
   ```

   An example output is as follows\.

   ```
   deployment.apps/coredns image updated
   ```

1. Check the container image version again to confirm that it was updated to the version that you specified in the previous step\.

   ```
   kubectl describe deployment coredns -n kube-system | grep Image | cut -d ":" -f 3
   ```

   An example output is as follows\.

   ```
   v1.10.1-eksbuild.3
   ```