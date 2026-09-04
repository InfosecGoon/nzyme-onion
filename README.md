# nzyme-onion

[Nzyme](https://www.nzyme.org) is a free and open platform for detecting physical, close-proximity attempts to compromise your network infrastructure. It leverages Wi-Fi, Bluetooth, and Ethernet traffic to detect these events, and its findings can be tied into [Security Onion](https://www.securityonion.com) to alert on threats like rogue access points or hidden Bluetooth trackers as soon as they appear.

This repository is intended to support the talk "Integrating Nzyme and Security Onion" at the 2026 Security Onion Conference. It contains the ingestion pipeline for Nzyme alerts and instructions for installing and using it in your Security Onion deployment.

# Instructions

1. Download the file "nzyme" from this repo and place it in `/opt/so/saltstack/local/salt/elasticsearch/files/ingest` on your Manager Node (or Standalone, in a single-node deployment).

2. Load the new ingest pipeline by running `sudo salt -C 'I@elasticsearch:enabled:true' state.apply elasticsearch queue=True` from the command line on that node.

3. In Elastic Fleet, open up the so-grid-nodes_general Agent Policy.

4. Click "Add Integration" and select "Custom UDP Logs" from the dropdown.

5. Enter the following configuration values:

   * Integration Name: nzyme
   * Listen Address: 0.0.0.0
   * Listen Port: 9001
   * Dataset name: nzyme
   * Ingest Pipeline: nzyme
   * Tags: nzyme
  
6. Click "Save Integration".

7. Create a [custom firewall rule](https://docs.securityonion.net/en/3/main/firewall/?h=firewall#creating-a-custom-host-group-with-a-custom-port-group) for port 9001/udp on the Manager or Standalone where you intend to receive the alert traffic.

8. Configure [Syslog Alerts on your Nzyme installation](https://docs.nzyme.org/configuration/alerting/action_types/) and send them to port 9001 on your Security Onion's IP address.

9. That should do it! If you want to confirm it's working, go into Hunt and search for `event.dataset:"nzyme"`.

BONUS: For a quick Dashboard, add this to soc > config > server > client > dashboards > queries in Configuration:

  `event.dataset: "nzyme" | groupby -pie "event_supertype" | groupby "details"`
