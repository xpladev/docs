---
title: Start the Light Client Daemon (LCD)
weight: 40
type: docs
---

The light client daemon (LCD) provides a REST-based adapter for the RPC endpoints, which also helps for decoding the Amino-encoded blockchain data into parseable JSON. This enables apps to communicate with a node through simple HTTP.

{{< alert >}}
**Note**

The XPLA Chain SDKs currently rely on an active connection to a running LCD server. If you need a dedicated connection for the SDKs, set up an LCD.
{{< /alert >}}

To enable the REST API and Swagger, and to start the LCD, complete the following steps:

1. Open `~/.xpla/config/app.toml`.

2. Locate the `API Configuration` section (`[api]`).

3. Change `enable = false` to `enable = true`.

    ```toml
    # Enable defines if the API server should be enabled.
    ```

4. Optional: To enable Swagger, change `swagger = false` to `swagger = true`.

    ```toml
    # Swagger defines if swagger documentation should automatically be registered.
    swagger = true
    ```
 5. Restart.

Once restarted, the LCD will be available.

For more information about the XPLA Chain REST API endpoints, see the [Swagger documentation](https://cube-lcd.xpla.dev/swagger/).

For more information on configuring `app.toml`, see [Configure General Settings]({{< ref "configure-general-settings" >}}).
