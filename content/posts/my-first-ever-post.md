---
title: 'Custom SSL Certificates on Coolify'
draft: false
---

# Custom SSL Certificates on Coolify

Written by Ted
ref: https://t3d.uk/blog/custom-ssl-certificates-on-coolify

Coolify is a Vercel replacement, and supports bringing your own infrastructure. This mean you can host as many resources are you like, without paying anyone anything, apart the service provider the server costs. SSL certificates give your websites a secure connection using https. Cloudflare is a Content Delivery Network, and can accelerate your website.

## Requirements 
- Domain is on Cloudflare
- Coolify instance

## Generating the SSL certificate
We will use acme.sh to generate the certificate. Once SSHed into the server, as root, run this command to install it.

```
curl https://get.acme.sh | sh
```

Once it has installed, **you will need to restart the session**. acme.sh will be added to the path.

You need to get a Cloudflare Token, which has permission to edit the DNS in your domain's zone. Once you have it, export it in the server like this.

```
export CF_Token="token"
```

Now we can generate the SSL certificate. **Remember** to replace domain.com with your domain.

```
acme.sh --issue --dns dns_cf --ocsp-must-staple --keylength 4096 -d domain.com -d '*.domain.com'
```

It may take a few minutes. Once it is done it will give you four paths, each to the different parts of the SSL certificate.

## Apply SSL Certificate
Let's create a folder to hold the SSL certificate inside the area which the proxy can access.

```
mkdir /data/coolify/proxy/certs
```

Let's now move the required parts of the SSL certificate into this new folder. You can use commands like cp to copy the files, or if the certificate was made on a different server, or your local computer, you can upload it with ftp or scp. You only need to copy the .cer and .key - the intermediate and full chain isn't needed.

Now, let's apply the configuration in Coolify. Head to Servers > localhost > Proxy > Dynamic Configuration. Create a new configuration called cert.yaml. Then paste the following into the configuration.

```
tls:
  certificates:
    -
      certFile: /traefik/certs/domainName.cer
      keyFile: /traefik/certs/domainName.key
```
## That's all
Once you restart your proxy, all resources and Coolify will use the wildcard SSL certificate if the domain is set to the one used in the SSL certificate.







