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
1. Subpaths in this subdomain may point to different services `svc1` and `svc2`
    - `svc1` and `svc2` are in separate deployments
1. The ingress is accessible through a public URL and secured with TLS

## Description of the setup

...
