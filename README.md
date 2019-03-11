# ssl-testing-with-docker

Test a SSL certificate and chain works fully, locally with nginx or Apache TomCat.

## Getting started

### Get your certificate chain ready

You should have `.crt` certificates likely in separate files, you'll need to create a single certificate chain file from these files to use it with nginx.

If you have all four files separated then use this command

```bash
cat sample_site_com.crt SectigoRSADomainValidationSecureServerCA.crt USERTrustRSAAddTrustCA.crt AddTrustExternalCARoot.crt > sample_site_full_chain.crt
```

If you have 2 files then use this command

```bash
cat sample_site_com.crt sample_site_com.ca-bundle > sample_site_full_chain.crt
```

[More information on certificate chains](https://medium.com/two-cents/certificate-chain-example-e37d68c3a3f0)

### Get your private.key file ready

When you created first created your `.csr` file a `.key` file was saved in the same folder. This is the file you'll need.

If you haven't created a CSR yet then you likely haven't created any certificates either, the command to create a CSR is shown for informational purposes only.

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
```

### Test your SSL certificate works

#### nginx

_All references to `help.paulsness.com` in this README should be replaced with your own site host._

##### Modify hosts file

For Mac you need to edit the file `/etc/hosts` and redirect traffic from `help.paulsness.com` to `127.0.0.1`.

##### Copy your certificate files

- `nginx/certs/sample.site.full_chain_cert.crt` - Copy the contents of your full chain cert here
- `nginx/certs/sample.site.sample.site.private.key` - Copy the contents of your private.key here

##### Get terminal running with Nginx

```bash
cd nginx

# Replace the place holder in the nginx conf with your own host
PLACEHOLDER_HOST='<YOUR_HOST_HERE>'
HOST='help.paulsness.com'
sed -i sample.site.conf "s,${PLACEHOLDER_HOST},${HOST},g"

# Build
docker build . -t ssl-test-nginx

# Run nginx
docker run -it --rm \
 -p 80:80 \
 -p 443:443 \
 -v $(pwd)/certs:/etc/nginx/certs \
 -v $(pwd)/sample.site.conf:/etc/nginx/conf.d/sample.site.conf \
 ssl-test-nginx \
 /bin/sh
```

##### Start Nginx once inside terminal

```bash
nginx -g "daemon off;"
```

##### Check SSL is working

You should run this command and check that no errors are shown at all you should see `Verify return code: 0 (ok)`.

```bash
openssl s_client -connect help.paulsness.com:443 -debug
```

[![See screenshot](readme-assets/open-ssl-test.png?raw=true)]

##### Remove docker image from system

You'll probably want to stop the container and remove the docker image to cleanup your system after running this test.

```bash
docker stop $(docker ps -q --filter ancestor=ssl-test-nginx )
docker rmi ssl-test-nginx
```
