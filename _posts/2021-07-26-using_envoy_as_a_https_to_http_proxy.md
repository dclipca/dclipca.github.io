# Using Envoy as a HTTPS to HTTP proxy (7X Faster than Nginx)

Let's create a folder:
```
mkdir envoy
```
Enter that folder:
```
cd envoy
```

To use HTTPS, you would need a SSL certificate. The simplest way to generate a private key and self-signed certificate for localhost is with this openssl command:

```
openssl req -x509 -out localhost.crt -keyout localhost.key \
  -newkey rsa:2048 -nodes -sha256 \
  -subj '/CN=localhost' -extensions EXT -config <( \
   printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```

After running the command above, you should get two files, ```localhost.crt``` and ```localhost.key```. You will have to trust those certificates from the Keychain (on macOS).

Create a ```envoy.yaml``` file with the following content:
```
static_resources:
  listeners:
    - address:
        socket_address:
          address: 0.0.0.0
          port_value: 443
      filter_chains:
        - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                codec_type: AUTO
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: app
                      domains:
                        - "*"
                      routes:
                        - match:
                            prefix: "/"
                          route:
                            cluster: service-http
                http_filters:
                  - name: envoy.filters.http.router
          transport_socket:
            name: envoy.transport_sockets.tls
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
              common_tls_context:
                tls_certificates:
                  - certificate_chain: { filename: "./localhost.crt" }
                    private_key: { filename: "./localhost.key" }
  clusters:
    - name: service-http
      type: STRICT_DNS
      lb_policy: ROUND_ROBIN
      load_assignment:
        cluster_name: service-http
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: 0.0.0.0
                      port_value: 3000

```

Considering that you have Envoy installed (run ```brew update && brew update && brew install envoy``` if not), you should run the following command in the envoy folder:
``` envoy -c envoy.yaml```
