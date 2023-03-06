# gc-syslog-tunnel
Create a stable tunnel from Centra Management through an Aggregator to reach a remote syslog server
Usage:
1. Upload the file to the management server
2. make the file executable [chmos +x gc-syslog-tunnel]
3. run the installation [./gc-syslog-tunnel install]
4. to uninstall run [./gc-syslog-tunnel uninstall]

This solution solves the issue that there is no build in function in centra to export network log as syslog via an aggregator.
Customers wanting to receive network log via syslog from a SaaS management require to expose their internal syslog server over the internet Many customers do not want to expose their internal syslog servers to any Internet source.
The script installs a systemd service which opens a listener on a given port and address on the management server and tunnels all connections from this port via the given aggregator, using ssh port forwarding via the aggregator to a given syslog server.
For the ssh connection it uses the same function the gcsh command uses. There is no direct ssh (port tcp/22) connecting from the management to the aggregator needed.
There are two elements which ensure a sable connection:
1. systemd monitors the processes and restarts them if they crash. The systemd service file can be found at /etc/systemd/system/gc-syslog-tunnel.service.
2. the script contains the autossh binady which it isnatlls as /usr/local/lib/autossh/autossh. Instead of ssh directly we call autossh as a wrapper for ssh which monitors the ssh connection via port 12345 and restarts the connection when it crashed.

The solution is tested on Centra up to v45. It is not tested on SaaSv2 and most likely will not work there.
Load considerations:
The reason why network log should not be exported via the aggregator was the processing tone in the exported on the aggregator would create too much load on the aggregator.
In this solution the network log does not get processes on the aggregator and as of such the load it produces is very low and does not increase much with the since of the network log. It only adds some to the network bandwidth. So be cautious for aggregators which have bandwidth limitations on the aggregator’s connections.

TODO:
1. adopt for SaaSv2. Could be run on the script server (not tested) but would not be surviving a restart of the script server pod as this would start the pod from the initial image.
2. change the systemd service to a generic one with the configuration in /etc/defaults/gc-syslog-tunnel
