# ssl-testing-with-docker

Test an SSL certificate and chain works entirely, locally with Nginx or Apache TomCat.

## Getting started

### Get your certificate chain ready

You should have `.crt` certificates likely in separate files, you'll need to create a single certificate chain file from these files to use it with nginx.

If you have all four files separated, then use this command

```bash
cat sample_site_com.crt SectigoRSADomainValidationSecureServerCA.crt USERTrustRSAAddTrustCA.crt AddTrustExternalCARoot.crt > sample_site_full_chain.crt
```

If you have two files, then use this command

```bash
cat sample_site_com.crt sample_site_com.ca-bundle > sample_site_full_chain.crt
```

[More information on certificate chains](https://medium.com/two-cents/certificate-chain-example-e37d68c3a3f0)

### Get your private.key file ready

When you created first created your `.csr` file a private `.key` file was saved in the same output directory. This is your private key.

If you haven't created a CSR yet then you likely haven't created any certificates either, the command to create a CSR is shown for informational purposes only.

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
```

[More on private keys here](https://www.digicert.com/blog/where-is-your-private-key/)

### Test your SSL certificate works

_All references to `help.paulsness.com` in this README should be replaced with your own site's host._

#### Modify hosts file

For Mac, you need to edit the file `/etc/hosts` and redirect traffic from `help.paulsness.com` to `127.0.0.1`.

#### Copy your certificate files

- `certs/sample.site.full_chain_cert.crt` - Copy the contents of your full chain cert here
- `certs/sample.site.sample.site.private.key` - Copy the contents of your private.key here

#### Launch the server with your certificates locally

##### Nginx

Get terminal running with Nginx

```bash
# Replace the place holder in the nginx conf with your own host
PLACEHOLDER_HOST='<YOUR_HOST_HERE>'
HOST='help.paulsness.com'
sed -i "" "s,${PLACEHOLDER_HOST},${HOST},g" nginx/sample.site.conf

# Build
docker build nginx -t ssl-test-nginx

# Run nginx
docker run -it --rm \
 -p 80:80 \
 -p 443:443 \
 -v $(pwd)/certs:/etc/nginx/certs \
 -v $(pwd)/nginx/sample.site.conf:/etc/nginx/conf.d/sample.site.conf \
 ssl-test-nginx \
 /bin/sh
```

Start Nginx once inside the terminal

```bash
nginx -g "daemon off;"
```

##### TomCat

```bash
# Build tomcat docker image
docker build tomcat -t ssl-test-tomcat

# Run Tomcat server and terminal
docker run -it --rm \
  -p 443:8443 \
  -v $(pwd)/tomcat/server.xml:/usr/local/tomcat/conf/server.xml \
  -v $(pwd)/certs/sample.site.full_chain_cert.crt:/usr/local/tomcat/conf/ssl.crt \
  -v /$(pwd)/certs/sample.site.private.key:/usr/local/tomcat/conf/ssl.key \
  ssl-test-tomcat \
  /bin/sh
```

Start TomCat server once inside the terminal

```bash
catalina.sh run
```

#### Check SSL certificates are valid after launching

You should run this command in a new terminal and check that there are no errors. You should see `Verify return code: 0 (ok)`.

```bash
openssl s_client -connect help.paulsness.com:443 -debug
```

![See screenshot](readme-assets/open-ssl-test.png?raw=true)

##### Remove docker image from system

You'll probably want to stop the container and remove the docker image to clean up your system after running this test.

```bash
# Nginx
docker stop $(docker ps -q --filter ancestor=ssl-test-nginx )
docker rmi ssl-test-nginx

# TomCat
docker stop $(docker ps -q --filter ancestor=ssl-test-tomcat )
docker rmi ssl-test-tomcat
```
