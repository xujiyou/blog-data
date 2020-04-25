# Header 清理

出于安全原因，Envoy将根据请求是内部请求还是外部请求来“清理”各种传入的HTTP Header，这些 Header 另作他用。清理操作取决于 Header，可能会添加，删除或修改。

最终，是将请求视为内部请求还是外部请求，由x-forwarded-for标头控制（请仔细阅读链接的部分，因为Envoy填充标头的方式很复杂，并取决于use_remote_address设置）

另外，internal_address_config设置可用于配置内部/外部确定。

Envoy可能会清理以下 Header：

- [x-envoy-decorator-operation](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-decorator-operation)
- [x-envoy-downstream-service-cluster](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-downstream-service-cluster)
- [x-envoy-downstream-service-node](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-downstream-service-node)
- [x-envoy-expected-rq-timeout-ms](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-expected-rq-timeout-ms)
- [x-envoy-external-address](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-envoy-external-address)
- [x-envoy-force-trace](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-envoy-force-trace)
- [x-envoy-internal](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-envoy-internal)
- [x-envoy-ip-tags](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ip_tagging_filter#config-http-filters-ip-tagging)
- [x-envoy-max-retries](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-max-retries)
- [x-envoy-retry-grpc-on](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-retry-grpc-on)
- [x-envoy-retry-on](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-retry-on)
- [x-envoy-upstream-alt-stat-name](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-upstream-alt-stat-name)
- [x-envoy-upstream-rq-per-try-timeout-ms](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-upstream-rq-per-try-timeout-ms)
- [x-envoy-upstream-rq-timeout-alt-response](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-upstream-rq-timeout-alt-response)
- [x-envoy-upstream-rq-timeout-ms](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-upstream-rq-timeout-ms)
- [x-forwarded-client-cert](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-forwarded-client-cert)
- [x-forwarded-for](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-forwarded-for)
- [x-forwarded-proto](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-forwarded-proto)
- [x-request-id](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-x-request-id)