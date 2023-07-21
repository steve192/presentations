These are sample applications to demonstrate treafik on docker


Install traefik by 
```
docker compose -f traefik-setup/docker-compose.yml up -d
```


Install applications by 
```
docker compose -f application/docker-compose.yml up -d
```

You can vary different options by editing application/docker-compose:

### sampleapp
- Hosts sample app under app.localhost (can be enteres in browser)
### securedapp
- Hosts sample app under secure.localhost
- Basic auth with username: user password: password
- Uncomment additional labels to enable automatically add security headers

### ratelimitedapp
- Hosts sample app under ratelimit.localhost
- Requets are rate limited by 1 / second
- Uncomment additional labels (and comment in treafik.http.routers.ratelimitedapp.middlewares=ratelimit1) to only enable rate limiting to ratelimit.localhost/api