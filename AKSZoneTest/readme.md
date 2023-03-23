# AKS Zone Test

[![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FScottHolden%2FAzureGym%2Fmain%2FAKSZoneTest%2F_generated%2Fdeploy.json)  
`Estimated deployment time: 8 minutes`  
**Click 'Deploy to Azure' to open the Azure Portal with this template loaded!**

## Description:
This is a small test scenario to show routing between an application deployed to multiple AZ's.

**While you can deploy to regions without AZ's, the tests will not work as designed**

## Resources ([Bicep Template](deploy.bicep))

The template deploys the following resources:
 - VNet & Log Analytics
 - AKS Cluster with 2 Node Pools (System + User) 
 - Azure Container Registry (with build task for Dockerfile in src)
 - Kube deployement & services (using the image built above) deployed 4 times, set up so one points to another ( 0 => 1 => 2 => 3 )

![Azure Resources](_media/azure-resources.png)
![Azure Resources](_media/aks-deployments.png)

## Service ([ASP.NET Core, .NET 7](src/))

The service is designed to show details of where it is hosted, and then make a call to the next service in the chain.  
(*Note: The latency is shown as round trip, eg, the first "Called next hop in Xms" represents the time it took from start of request till complete response.*)

![Http Response](_media/example-response.png)

## Test Script ([Powershell](quick-test.ps1))
Also included is a powershell test script that you can use to make multiple calls to the service exposed and gather details about the number of zone hops and latency.

![Powershell Output](_media/ps-output.png)

## Using Linkerd to route requests within the zone
Service Mesh can help in traffic management inside your cluster. Linkerd uses an algorithm called EWMA, or exponentially weighted moving average, to automatically send requests to the fastest endpoints. This load balancing can improve end-to-end latencies.

### Install Linkerd 
Please use the steps highlighted to install linkerd in your aks cluster - https://linkerd.io/2.12/getting-started/. You will need to install the linkerd client, the crd and the control plane. Once installed, your linkerd checks should give all green. 

### Inject linkerd inside your deployments

```
kubectl get -n zonetest deploy -o yaml | linkerd inject - | kubectl apply -f -

```

Now run the powershell script again and you will see the following output

![Powershell Output](_media/linkerd-zonetest.png)

