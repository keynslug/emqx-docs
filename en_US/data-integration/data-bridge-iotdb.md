# Apache IoTDB

[Apache IoTDB](https://iotdb.apache.org/) is a high-performance and scalable time series database that is designed to handle massive amounts of time series data generated by various IoT devices and systems. By ingesting data into Apache IoTDB through the data bridge, you can forward data from other systems to Apache IoTDB for storage and analysis.

EMQX supports data integration with Apache IoTDB, allowing you to forward time series data to Apache IoTDB using their [REST API V2](https://iotdb.apache.org/UserGuide/Master/API/RestServiceV2.html).

{% emqxce %}
:::tip 

The Apache IoTDB bridge is an EMQX Enterprise Edition feature. EMQX Enterprise Edition provides comprehensive coverage of key business scenarios, rich data integration, product-level reliability, and 24/7 global technical support. Experience the benefits of this [enterprise-ready MQTT messaging platform](https://www.emqx.com/en/try?product=enterprise) today.
::: {% endemqxce %}

::: tip Prerequisites

- Knowledge about EMQX data integration [rules](./rules.md)

- Knowledge about [data bridges](./data-bridges.md)

- Basic knowledge of UNIX terminal and commands

:::

## Feature List

- [Connection pool](./data-bridges.md#connection-pool)
- [Async mode](./data-bridges.md#async-mode)
<!-- - [Batch mode](./data-bridges.md) -->
<!-- - [Buffer mode](./data-bridges.md) -->

## Quick Start Tutorial

This section introduces how to use the Apache IoTDB data bridge with a practical tutorial, covering topics like creating an Apache IoTDB instance, creating a data bridge, setting up a rule for forwarding data to the data bridge, and testing the data bridge and rule.

This tutorial assumes that you have an IoTDB instance running on the local machine. If you have Apache IoTDB running remotely, please adjust the settings accordingly.

### Start an Apache IoTDB Server

This section introduces how to start an Apache IoTDB server using [Docker](https://www.docker.com/). Make sure to have `enable_rest_service=true` in your IoTDB's configuration.

Run the following command to start an Apache IoTDB server with its REST interface enabled:

```bash
docker run -d --name iotdb-service \
              --hostname iotdb-service \
              -p 6667:6667 \
              -p 18080:18080 \
              -e enable_rest_service=true \
              -e cn_internal_address=iotdb-service \
              -e cn_target_config_node_list=iotdb-service:10710 \
              -e cn_internal_port=10710 \
              -e cn_consensus_port=10720 \
              -e dn_rpc_address=iotdb-service \
              -e dn_internal_address=iotdb-service \
              -e dn_target_config_node_list=iotdb-service:10710 \
              -e dn_mpp_data_exchange_port=10740 \
              -e dn_schema_region_consensus_port=10750 \
              -e dn_data_region_consensus_port=10760 \
              -e dn_rpc_port=6667 \
              apache/iotdb:1.1.0-standalone
```

You can find more information about running [IoTDB in Docker on Docker Hub](https://hub.docker.com/r/apache/iotdb).

### Create an Apache IoTDB Data Bridge

This section introduces how to create an EMQX data bridge to Apache IoTDB through Dashboard.

1. Go to the Dashboard, and click **Data Integration** -> **Data Bridge** from the left navigation menu.

2. Click **Create** on the top right corner of the page.

3. In the **Create Data Bridge** page, click to select **Apache IoTDB**, and then click **Next**.

4. Input a name for the data bridge. The name should be a combination of upper/lower case letters and numbers.

5. Input the connection information:
   * **Base URL**: Input `http://localhost:18080`, or the actual hostname/IP if the IoTDB server is running remotely.
   * **Username**: Input the IoTDB username; The default value is `root`.
   * **Password**: Input the IoTDB password; The default value is `root`.
   
6. Configure the **Payload Template**: The default value is an empty string, meaning the message payload will be forwarded as JSON-formatted text to IoTDB without modification.

   You can also define a custom message payload format using placeholders within the template to dynamically include data from the incoming MQTT messages. For example, if you want to include the MQTT message payload and its timestamp in the IoTDB message, you can use the following template:

   ```sql
   {"payload": "${data}", "timestamp": ${timestamp}}
   ```

   `${data}` and `${timestamp}` are placeholders and will be replaced by the actual values from the message when it is forwarded to the Apache IoTDB server. This template will produce a JSON-formatted message containing the payload and timestamp of the incoming MQTT message. 

7. Advanced settings (optional):  Choose whether to use **sync** or **async** query mode as needed.

8. Before clicking **Create**, you can click **Test Connectivity** to test that the bridge can connect to the Apache IoTDB server.

9. Click **Create** to finish the creation of the data bridge.

   A confirmation dialog will appear and ask if you like to create a rule using this data bridge, you can click **Create Rule** to continue creating rules to specify the data to be saved into Apache IoTDB. You can also create rules by following the steps in [Create Rules for Apache IoTDB Data Bridge](#create-a-rule-for-apache-iotdb-bridge).

Now the Apache IoTDB data bridge should appear in the data bridge list (**Data Integration** -> **Data Bridge**) with Resource Status as Connected. 

### Create a Rule for Apache IoTDB Data Bridge

You can continue to create a rule to forward data to the new Apache IoTDB bridge.

1. Go to the EMQX Dashboard, and click **Integration -> Rules**.

2. Click **Create** on the top right corner of the page.

3. Input a rule ID, for example, `my_rule`.

4. Input the following statement in the SQL editor, which will forward the MQTT messages matching the topic pattern `t/#`:

   ```
   SELECT
     *
   FROM
     "t/#"

   ```

5. Click the **Add Action** button, select **Forwarding with Data Bridge** from the dropdown list, and then select the data bridge you just created under **Data Bridge**.

6. Click the **Add** button to finish the setup.

7. Click the **Create** button at the bottom of the page to finish the setup.

Now a rule to forward data to Apache IoTDB via the data bridge is created. You can click **Data Integration** -> **Flows** to view the topology. It can be seen that the messages under the topic `t/#` are sent and saved to Apache IoTDB.

### Test the Data Bridge and Rule

You can use the built-in WebSocket client in the EMQX dashboard to test your rule and bridge.

1. Click **Diagnose** -> **WebSocket Client** in the left navigation menu of the Dashboard.

2. Fill in the connection information for the current EMQX instance. 

   - If you run EMQX locally, you can use the default value.
   - If you have changed EMQX's default configuration. For example, the configuration change on authentication can require you to type in a username and password.

3. Click **Connect** to connect the client to the EMQX instance.

3. Scroll down to the publish area and type in the following:
   * **Topic**:`t/test`
   * **Payload**: Make sure to select `JSON` as the payload encoding and then enter:
   ```json
   {
     "measurement": "temp",
     "data_type": "FLOAT",
     "value": "37.6",
     "device_id": "root.sg27"
   }
   ```
   * **QoS**: `2`
   
4. Click **Publish** to send the message.

If the data bridge and rule are successfully created, the message should have been published to the specified time series table in the Apache IoTDB server. You can check this using IoTDB's command line interface. If you're using it from docker as above you can connect the server by using this command from your terminal:

```shell
    $ docker exec -ti iotdb-service /iotdb/sbin/start-cli.sh -h iotdb-service
```

Then, from within this console, type the following:
```sql
IoTDB> select * from root.sg27
```
And you should see the data printed as follows:
```
+------------------------+--------------+
|                    Time|root.sg27.temp|
+------------------------+--------------+
|2023-05-05T14:26:44.743Z|          37.6|
+------------------------+--------------+
```