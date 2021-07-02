# DomainNameThreatFeed


Pulling a DNS-based threat feed into Calico Enterprise: 
```
cat << EOF > isc-suspicious-domains.yaml
apiVersion: projectcalico.org/v3
kind: GlobalThreatFeed
metadata:
  name: internet-storm-center
  labels:
    threat-feed: suspiciousdomains
spec:
  content: DomainNameSet
  pull:
    http:
      url: https://isc.sans.edu/block.txt
EOF      
```

```
kubectl apply -f isc-suspicious-domains.yaml
```

Pulling an IP-based threat feed into Calico Enterprise:
```
cat << EOF > feodo-tracker.yaml
apiVersion: projectcalico.org/v3
kind: GlobalThreatFeed
metadata:
  name: feodo-tracker
spec:
  content: IPSet
  pull:
    http:
      url: https://feodotracker.abuse.ch/downloads/ipblocklist.txt
  globalNetworkSet:
    labels:
      threat-feed: feodo
EOF
```

Create a network policy for the domain name threat feed
```
cat << EOF > domain-feed-policy.yaml
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: security.domain-feed
spec:
  tier: security
  order: 210
  selector: projectcalico.org/namespace != "acme"
  namespaceSelector: ''
  serviceAccountSelector: ''
  egress:
    - action: Deny
      source: {}
      destination:
        selector: threatfeed == "suspiciousdomains"
  doNotTrack: false
  applyOnForward: false
  preDNAT: false
  types:
    - Egress
EOF
```

```
kubectl apply -f domain-feed-policy.yaml
```

Strangely-formatted threat feed of domain names and IP addresses:

```
apiVersion: projectcalico.org/v3
kind: GlobalThreatFeed
metadata:
  name: russian-tracker
spec:
  content: IPSet
  pull:
    http:
      url: https://urlhaus.abuse.ch/feeds/country/RU/
  globalNetworkSet:
    labels:
      threat-feed: ru
```
