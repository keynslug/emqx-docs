# File Transfer over MQTT

While core MQTT protocol features are sufficient for most IoT applications and scenarios, there are some use cases that require MQTT client devices to have an ability to transfer large files to a broker. Examples of such use cases are:
* A smart camera need to transfer a video stream to a cloud storage.
* Log files from a vehicle need to be sent to an analytics server.
* Warehouse robots need to send captured images to a cloud server for auditing.

For such use cases, client devices usually had to resort to using other protocols such as HTTP or FTP. This approach has several disadvantages, namely:
* Need to implement other protocols, with their own security and authentication mechanisms, with complex state that should be maintained on the client device.
* Need to have a set of separate services to handle file transfers.

In order to alleviate these extra needs, EMQX provides a _File Transfer over MQTT_ feature that allows MQTT clients to transfer files to a broker using the exact same MQTT protocol that they use for other operations. MQTT specification does not define any standard way to transfer files. Instead, EMQX defines and implements a simple, application-level protocol on top of MQTT that facilitates file transfers, while relieving the client devices from much of the complexity.
