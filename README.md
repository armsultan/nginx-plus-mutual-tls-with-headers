# Instructions
 1. Place the following files in the directories
    * Download your Keys and Certificates from the [customer portal](https://cs.nginx.com/) or from
    an activated evaluation, `nginx-repo.crt` and `nginx-repo.crt` into `etc/ssl/nginx/`
    * Your Nginx conigurations in `etc/nginx/` (and sub directories like `etc/nginx/conf.d` and `etc/nginx/stream.conf.d`)
    * If you are doing an "offline" install of Nginx Plus, place the Nginx Plus binaries, `.rpm` or `.deb` files, in the `install` folder
 2. Copy a `Dockerfile` for your preferred linux distro from the `Dockerfiles` folder into the root folder
 3. Edit (add or uncomment) the `Dockerfile` to include any necessary commands to install addtional modules
 4. Make sure the `nginx.conf` (`/etc/nginx/nginx.conf`) includes the necessary `load_modules` directives to load addtional modules installed
 5. Build an image from your Dockerfile:
    ```bash
    # Run command from the folder containing the `Dockerfile`
    docker build -t nginx-plus-mtls .
    ```
 6. Start the Nginx Plus container, e.g.:
    ```bash
    # Start a new container and publish container ports 80, 443 and 8080 to the host
    docker run -d -p 80:80 -p 443:443 -p 8080:8080 nginx-plus-mtls
    ```

    **To mount local volume:**

    ```bash
    docker run -d -p 80:80 -p 443:443 -p 8080:8080 \
      -v $PWD/etc/nginx/conf.d:/etc/nginx/conf.d \
      -v $PWD/etc/nginx/nginx.conf:/etc/nginx/nginx.conf \
      nginx-plus-mtls
    ```

 7. If the provided file, `status_api.conf` is loaded then the NGINX Plus live activity monitoring dashboard can be
    accessed from [`http://localhost:`](http://localhost:8080)
 8. For troubleshooting or quick nginx configuration edits, you can ssh into container: `docker exec -it <container name> /bin/bash`

 9. Run docker logs to see the headers inserted to the backend:

   ```bash
   docker ps # get Docker IDs of running containers
   sudo docker logs -f [CONTAINER ID]
   ```
   
## Test

#### Without Client certificate:

```bash
curl -v -k  --Header 'Host: www.example.com' https://localhost
```

#### With Client certificate:


```bash
# client_1
curl -v -k \
  --key ./client_certs/client_1/client_1.key \
  --cert ./client_certs/client_1/client_1.crt \
  --Header 'Host: www.example.com' \
  https://localhost

# client2
curl -v -k \
  --key ./client_certs/client_2/client_2.key \
  --cert ./client_certs/client_2/client_2.crt \
  --Header 'Host: www.example.com' \
  https://localhost

# client_3
curl -v -k \
  --key ./client_certs/client_3/client_3.key \
  --cert ./client_certs/client_3/client_3.crt \
  --Header 'Host: www.example.com' \
  https://localhost
  ```


  curl -v -k \
  --key ./client_certs/client_2/client_2.key \
  --cert ./client_certs/client_2/client_2.crt \
  --cacert ./certificate_authority/ca.crt \
  --Header 'Host: www.example.com' \
  https://localhost