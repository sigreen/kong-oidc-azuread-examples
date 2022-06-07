Setup Kong OIDC plugin with AzureAD
===========================================================

This example walks through the setup and configuration of both Azure AD and the Kong OIDC plugin.  It builds upon the authoratative documentation from Kong, given [here](https://docs.konghq.com/gateway/latest/developer-portal/administration/application-registration/azure-oidc-config/#main).

## Prerequisites
1. Azure Credentials
2. Working Kong cluster with RBAC enabled.
3. `portal_app_auth: external-oauth2` was set in the values.yaml during helm installation

## Procedure - Azure AD Setup

1. Via the Azure Web Portal, navigate to `App registrations`.  Create a `New registration` and select the following details:

* Name: alpha-team-oidc
* Supported account types: Accounts in any org directory
*  Redirect URI: Web, https://<kong-proxy-hostname>/httpbin-azure

2. Take Note of the following credentials and paste them into notepad

* Application (client) ID
* Directory (tenant) ID

3. Click the `Certificates & secrets` link then click `New client secret`.  Give the description `alpha-team-oidc-cert` and click `Add`.

4. Copy and Paste the `Value` - this will actually be your Client Secret going forward, into your notepad.

## Procedure - Kong Setup

1. Setup an echo-server application to demonstrate how to use the Kubernetes Ingress Controller:

```
kubectl apply -f https://bit.ly/echo-service -n kong
```

2. Create a Basic Proxy

```
kubectl apply -f crd/01-ingress.yml -n kong
```

This will apply the Ingress object to the Kubernetes API which will be intercepted by the Kong Ingress Controller and applied to Kong config.

3. Replace the values in `crd/openid-connect-plugin.yml` with the correct values taken from Azure in your notepad.

4. Enable the OIDC and portal registration plugins as `kongplugin` in k8s:

We first apply the KongPlugin CRD

```
kubectl apply -f crd/openid-connect-plugin.yml -n kong
kubectl apply -f crd/portal-app-plugin.yml -n kong
```

5. Then, apply it to an ingress (Route or Routes) by annotating the ingress rule

```
kubectl apply -f crd/portalreg-openid-ingress.yml -n kong
```
