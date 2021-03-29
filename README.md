# Crash in 1.17.1 with unavailable ratelimit cluster

I believe there is a regression in Envoy 1.17.x where if ratelimiting is
configured for a cluster but the rate limit service is unavailable then a
request to that cluster will cause Envoy to crash. This does not happen in
1.16.0.

This demonstrates this working in 1.16.0 and causing a crash in 1.17.1.

## Behaviour in 1.16.0 (no crash)

```console
# optionally rm all docker-compose containers, just in case
docker-compose -f docker-compose-1.16.yaml rm -f && docker-compose -f docker-compose-1.17.1.yaml rm -f
docker-compose -f docker-compose-1.16.yaml up 
curl -v http://localhost:8888/status/200
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8888 (#0)
> GET /status/200 HTTP/1.1
> Host: localhost:8888
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< server: envoy
< date: Mon, 29 Mar 2021 10:02:51 GMT
< content-type: text/html; charset=utf-8
< access-control-allow-origin: *
< access-control-allow-credentials: true
< content-length: 0
< x-envoy-upstream-service-time: 6
<
* Connection #0 to host localhost left intact
```

## Behaviour in 1.17.1 (Envoy crashes!)

```console
# optionally rm all docker-compose containers, just in case
docker-compose -f docker-compose-1.16.yaml rm -f && docker-compose -f docker-compose-1.17.1.yaml rm -f
docker-compose -f docker-compose-1.17.1.yaml up
curl -v http://localhost:8888/status/200
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8888 (#0)
> GET /status/200 HTTP/1.1
> Host: localhost:8888
> User-Agent: curl/7.54.0
> Accept: */*
>
* Empty reply from server
* Connection #0 to host localhost left intact
curl: (52) Empty reply from server
```

Envoy will now have crashed:

```
envoy_1  | Our FatalActions triggered a fatal signal.
envoycrashdemo_envoy_1 exited with code 139
```