# Streaming Telemetry Lab

> [!CAUTION]
> Before proceeding, please destroy all other labs to free up the memory  
> `clab des -c -a` - destroys all running labs and removes their respective lab dirs

When pulling together what we have learned so far, we can build a lab that embodies a lot of what containerlab has to offer. One of the labs that we've built to demonstrate the power of containerlab is the [Streaming Telemetry Lab](https://github.com/srl-labs/srl-telemetry-lab).

To deploy this lab we will use another neat containerlab feature - the ability to deploy a lab by specifying lab's remote URL. For the labs stored as a repo on GitHub we could even use the shorthand syntax:

```bash
cd ~
containerlab deploy -t srl-labs/srl-telemetry-lab
```

This will pull down the repository and deploy the lab right away.
The deployment will take a couple of minutes because several images will be downloaded during the process.

## Exploring the topology file

The streaming telemetry lab' [topology file](https://github.com/srl-labs/srl-telemetry-lab/blob/main/st.clab.yml) is worth a closer look. It features:

- customization of the management network
- use of `defaults` and `kinds` sections to simplify the topology file
- static management IPs for consistency
- combination of network OSes and "regular" containerized workloads like iperf clients, streaming telemetry and logging stack
- use of the bind mounts to load the configuration files
- use of env vars to parametrize the started containers
- `exec` command to run commands in the started containers
- port exposure to the host to make the lab services accessible from the outside
- using of `group` parameter to influence lab nodes ordering in the graph products

As you can see, this topo file is not a joke, and once the deployment finishes, you might want to understand what exactly was deployed and how the topology is structured.    

## Streaming Telemetry stack

As the lab name suggests, telemetry is at its core. The following telemetry stack is used in this lab:

| Role                | Software                              |
| ------------------- | ------------------------------------- |
| Telemetry collector | [gnmic](https://gnmic.openconfig.net) |
| Time-Series DB      | [prometheus](https://prometheus.io)   |
| Visualization       | [grafana](https://grafana.com)        |

Using containerlab's ability to expose ports of the containers to the host, telemetry services are available on the host machine, Access Grafana:

* Grafana: <http://localhost:3000>. Anonymous access is enabled; no credentials are required. If you want to act as an admin, use `admin/admin` credentials.


To run traffic between the nodes, leverage `traffic.sh` control script.
To start the traffic:

* `bash traffic.sh start all` - start traffic between all nodes

As a result, the traffic will be generated between the clients and the traffic rate will be reflected on the grafana dashboard.

<https://github.com/srl-labs/srl-telemetry-lab/assets/5679861/158914fc-9100-416b-8b0f-cde932895cec>

For more details refer to the [Nokia SR Linux Streaming Telemetry Lab](https://github.com/srl-labs/srl-telemetry-lab/tree/main).


## Containerlab and graphing options

When your topology has 3-5 nodes and a few links, you can easily visualize the topology in your head, but once your lab grows bigger, with more nodes involved and more links created, it becomes harder to remember what is what and where.

To help you with visualizing the topology, containerlab provides a few graphing options (<https://containerlab.dev/cmd/graph/>) that we will explore briefly.

### Web graph

After you deployed the lab, run the `graph` command:

```bash
cd ~/srl-telemetry-lab/
clab graph
```

The local web server provided by containerlab will be welcoming you at `http://<vm_ip_address>:50080` where `<vm_ip_address>` is your VM IP address. The web server will render the deployed topology in an interactive graph that you can sort vertically or horizontally.

The `group` options on the node level help you to order the lab nodes in the graph.

### Drawio export

The web graph is great, but has a number of limitations:

- Not editable
- Opinionated
- Only one ”pane” - physical

To combat these limitations, another project has been created in the containerlab ecosystem - [clab-io-draw](https://github.com/srl-labs/clab-io-draw).

It allows to generate a drawio topology from a containerlab topology and does so with style!

```bash
clab graph --drawio -t st.clab.yml --drawio-args "--theme nokia_modern"
```
The newly created drawio file can be found in the same folder
```bash
[*]─[vm16]─[/]
└──> cd ~/srl-telemetry-lab/

[*]─[vm16]─[~/srl-telemetry-lab]
└──> ls
LICENSE  README.md  clab-st  configs  st.clab.drawio  st.clab.yml  st.clab.yml.annotations.json  streaming-telemetry.drawio  traffic.sh
```

[clab-io-draw](https://github.com/srl-labs/clab-io-draw) has many neat options, so please go checkout the readme file at the projects' repo.

### 
