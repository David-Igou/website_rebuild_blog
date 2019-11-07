# ingress-basicauth

Here's a pretty easy example for adding basic password auth to a Kubernetes ingress

```
$ htpasswd -nb david stinkysocks
david:$apr1$dxwaFeYS$Tt3D4YsaFyja1W1zPPXUh0
$ echo -n `david:$apr1$dxwaFeYS$Tt3D4YsaFyja1W1zPPXUh0` |base64
ZGF2aWQ6JGFwcjEkZHh3ekZlFUMEzULTRAc2FGeGREATSWORDxelBQWFVoMA==
```

Stick it in a secret

```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: ingress-auth
data:
  auth: ZGF2aWQ6JGFwcjEkZHh3ekZlFUMEzULTRAc2FGeGREATSWORDxelBQWFVoMA==
```

Add these annotations to your ingress:

```
    ingress.kubernetes.io/auth-secret: ingress-auth
    ingress.kubernetes.io/auth-type: basic
```

Works in my lab
