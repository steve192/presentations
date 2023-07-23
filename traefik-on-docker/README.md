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


### Metrics
- Uninstall traefik
```
docker compose -f traefik-setup/docker-compose.yml down
```
- Install traefik with metrics on
```
docker compose -f traefik-setup/docker-compose-with-metrics.yml up -d
```
- Access http://grafana.localhost
- Login with admin/admin
- Set new password
- Add new datasource InfluxDB (Administration > Datasources > Add > InfluxDB)
    - Query Language Flux
    - URL http://influxdb:8086
    - Disable basic auth
    - organisation=org
    - token=password
    - default bucket=bucket
    - save&test
- Import the sample dashboard traefik-setup/grafana-sample-dashboard.json from this repository
    - In grafana choose dashboards > new > import and paste the contents of grafana-sample-dashboard.json
    - Give it a name an associate with your newly created datasource
- Optional if you did not import the sample-dashboard: Add visualization (Home > Add > Visualization)
    - Select InfluxDB as datasource
    - Add flux query to show all requests to entrypoint (Visualization type timeseries)
    ```
        from(bucket: "bucket")
        |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
        |> filter(fn: (r) =>
            r._measurement == "traefik.entrypoint.requests.total"
        )
        |> group(columns: ["entrypoint", "method", "protocol"])
        |> drop(columns: ["_start", "_stop"])
    ```
    - Add flux query to show distribution of http codes (Visualization type Pie Chart)
    ```
    from(bucket: "bucket")
        |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
        |> filter(fn: (r) =>
            r._measurement == "traefik.entrypoint.requests.total" and
            r.entrypoint == "http"
        )
        |> group(columns: ["code"])
        |> count()
        |> drop(columns: ["_start", "_stop"])
        |> rename(columns: {_value: "http code"})
        
    ```
    - Add flux query to show average response times (90th percentile) (Visualization type timeseries)
    ```
    from(bucket: "bucket")
    |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
    |> filter(fn: (r) => r["_measurement"] == "traefik.entrypoint.request.duration")
    |> filter(fn: (r) => r["entrypoint"] == "http" and r["protocol"] != "websocket")
    |> filter(fn: (r) => r["_field"] == "p90")
    |> group()
    |> aggregateWindow(every: v.windowPeriod, fn: mean)
    |> map(fn: (r) => ({r with _value:  r._value * 1000.0 }))
    ```
    - Save