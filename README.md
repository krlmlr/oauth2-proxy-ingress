# oauth2-proxy-ingress

## Initialization

1. Start Kubernetes cluster, e.g. via <https://www.katacoda.com/courses/kubernetes/playground>
1. Clone this repository there: `git clone https:// :github.com/krlmlr/oauth2-proxy-ingress`
1. Init the ingress: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/cloud/deploy.yaml`

## Requirements

1. In the cluster, one subdomain is protected with authentication via [oauth2-proxy](https://github.com/oauth2-proxy/oauth2-proxy) with a stock image
1. The ingress for this subdomain is connected with Okta
    - Only one callback URL is configured in Okta
    - The Okta configuration is a throwaway one, we don't care too much about secrecy of client ID and client secret
    - Configure Okta Domain with Oauth2 Proxy with:
	    - clientId=0oa9ouy57yNog4AYr2p7
	    - client_secret=ZOzPBR7iAJYOx7E4vV4GWOvqMsF8LXnmqYTZem1Y
            - issuer=https://hudea.okta.com/oauth2/default 
	    - --validate-url=https://hudea.okta.com/oauth2/default/v1/introspect
1. Subpaths in this subdomain may point to different services `svc1` and `svc2`
    - `svc1` and `svc2` are in separate deployments
1. The ingress is accessible through a public URL and secured with TLS

## Description of the setup

1) Run Google Cloud Kubernetes Engine with HTTP LoadBalaner (https://cloud.google.com/kubernetes-engine/docs/quickstart)
- install GC SDK: https://cloud.google.com/sdk/docs/install

2) Authenticate with the GC CLI and export cluster config: 
- authenticate:
	gcloud auth login
- export kube config:
	gcloud container clusters get-credentials alx-cluster-1 --zone europe-west1-c # My test cluster alx-cluster-1 and zone

3) Deploy Ingress Controller (https://kubernetes.github.io/ingress-nginx/deploy/ https://cloud.google.com/community/tutorials/nginx-ingress-gke)
kubectl apply -f ingress-controller-deploy.yml
- check ingress controller: 
	kubectl -n ingress-nginx get svc -o wide -w ingress-nginx-controller

4) Deploy oauth2-proxy (https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview)
- Create deployment for oauth2-proxy:
	kubectl apply -f oauth2-proxy.yml
- Create oauth2-proxy service:
	kubectl apply -f oauth2-proxy-svc.yml
- Create oauth2-proxy ingress:
	kubectl apply -f oauth2-proxy-ing.yml

5) Deploy app service and test with oauth2-proxy:
- Create deployment for app:
	kubectl apply -f hi-alx.yml
- Create the app service:
	kubectl apply -f hi-alx-svc.yml
- Create Ingres for app (- with auth_request: https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview/#configuring-for-use-with-the-nginx-auth_request-directive)	
	kubectl apply -f hi-alx-ing.yml
		
6) If domain needs to resolve locally add Hostname directive on your local machine with hostname and ingress-nginx-controller EXTERNAL-IP
104.199.xx.xx test.domain.com
- test out: 
	http://hostname/
redirects to: IDP
	- if authentication is successful, client redirects back to the / app path with the oauth2-proxy cookie and authorization headers configured