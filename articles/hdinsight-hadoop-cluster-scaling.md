<properties
   pageTitle="Cluster Scaling in HDInsight | Azure"
   description="Change the number of data nodes in a cluster that is running on HDInsight without having to delete and recreate the cluster."
   services="hdinsight"
   documentationCenter=""
   authors="bradsev"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang=""
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="02/18/2015"
   ms.author="bradsev"/>

#Cluster Scaling in HDInsight

The cluster scaling feature allows you to change the number of data nodes used by a cluster that is running in HDInsight without having to delete and recreate the cluster. The operation can be performed using PowerShell, the HDInsight SDK, or from the Azure portal.

> [WACOM.NOTE] Only the Hadoop and Storm cluster types are supported in the current release. Support for HBase clusters will be added shortly. 

## Feature Details
This section describes the impact of changing the number of data nodes for each type of cluster supported by HDInsight:

* Hadoop
* Storm
* HBase 

## Hadoop 

### Adding Data Nodes
You can seamlessly increase the number of data nodes in a Hadoop cluster that is running without impacting any pending or running jobs. New jobs can also be submitted while the operation is in progress. Failures in a scaling operation are gracefully handled so that the cluster is always left in a functional state.

### Removing Data Nodes
When an Hadoop cluster is scaled down by reducing the number of data nodes, some of the services in the cluster are restarted. This causes all running and pending jobs to fail at the completion of the scaling operation. You can, however, resubmit the jobs once the operation is complete.

## Storm
You can seamlessly add or remove data nodes to your Storm cluster while it is running. But after a successful completion of the scaling operation, you will need to re-balance the topology.

Rebalancing can be accomplished in two ways using:

* Storm web UI
* CLI tool 

Please refer to the [Apache Storm documentation](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html) for more details.

The Storm Web UI is available on the HDInsight cluster:

![image1](./media/hdinsight-hadoop-cluster-scaling/StormUI.png)

Here is an example how to use the CLI command to rebalance the Storm topology:

	## Reconfigure the topology "mytopology" to use 5 worker processes,
	## the spout "blue-spout" to use 3 executors and
	## the bolt "yellow-bolt" to use 10 executors.

	$ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

##HBase
The cluster scaling operation is not supported for HBase type clusters at this time.

## Prerequisites:

* Only clusters with an HDInsight version 3.1.3 or higher are supported. If you are unsure of the version of your cluster, you can check the cluster version from the Azure Portal by clicking the HDInsight cluster name or by running the `Get-AzureHDInsightCluster –name <clustername>` command from Azure PowerShell.

* Azure PowerShell version 0.8.14 or higher is required to perform the operation from PowerShell. You can download the latest version of PowerShell from the Command-line tools section on the [Azure Downloads](http://azure.microsoft.com/downloads/) website. You can check on the Azure PowerShell version you have installed with the following command from a PowerShell window: `(get-module Azure).Version`

## How to use Cluster Scaling

### Azure Portal
The size of a running cluster can be changed from the **Scale** tab of Azure HDInsight cluster dashboard.

![](http://i.imgur.com/u5Mewwx.png)

### PowerShell
To change the Hadoop cluster size using PowerShell, run the following command from a client machine:

	Set-AzureHDInsightClusterSize -ClusterSizeInNodes <NewSize> -name <clustername>	

> [WACOM.NOTE] The client machine must have Azure PowerShell version 0.8.14 or higher installed to use this command.

### SDK
To change the Hadoop cluster size using the HDInsight SDK, use one of the following two methods: 

	ChangeClusterSize(string dnsName, string location, int newSize) 

or 

	ChangeClusterSizeAsync(string dnsName, string location, int newSize) 


Both synchronous and asynchronous versions of this method return [ClusterDetails](http://msdn.microsoft.com/library/microsoft.windowsazure.management.hdinsight.clusterdetails_properties.aspx) object.

Here is some sample code that shows how to use the synchronous version of this method:

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using System.Security.Cryptography.X509Certificates;
	using Microsoft.WindowsAzure.Management.HDInsight;
	using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning;

	namespace HDInsightClusterScaling
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            // Friendly name for the certificate your created earlier  
	            string certfriendlyname = "<CertificateFriendlyName>";     
	            string subscriptionid = "<SubscriptionID>";
	            string clustername = "<ClusterDNSName>";
	     		string location = "<ClusterLocation>”";
				int newSize = <NewClusterSize>;
	
	            // Get the certificate object from certificate store using the friendly name to identify it
	            X509Store store = new X509Store();
	            store.Open(OpenFlags.ReadOnly);
	            X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certfriendlyname);
	
	            // Create an HDInsightClient object
	            HDInsightCertificateCredential creds = new HDInsightCertificateCredential(new Guid(subscriptionid), cert);
	            var client = HDInsightClient.Connect(creds);
	
	            Console.WriteLine("Rescaling HDInsight cluster ...");
	
	            // Change cluster size
	     		ClusterDetails cluster = client.ChangeClusterSize(clustername, location, newSize);
	            
	            Console.WriteLine("Cluster Rescaled: {0} \n New Cluster Size = {1}", cluster.ConnectionUrl, cluster.ClusterSizeInNodes);
	            Console.WriteLine("Press ENTER to continue.");
	            Console.ReadKey();
	        }
	    }
	}


Please refer to the [Provision Hadoop clusters in HDInsight using custom options](http://azure.microsoft.com/documentation/articles/hdinsight-provision-clusters/) topic for more information on using HDInsight .NET SDK.