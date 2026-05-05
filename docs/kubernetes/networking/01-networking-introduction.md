# 1. Networking Introduction

!!! info "Source Attribution"

    The primary source and original content for this page originate from [the **Networking & Kubernetes - A Layered Approach** by James Strong and Vallery Lancey](https://www.oreilly.com/library/view/networking-and-kubernetes/9781492081647/). Please refer to [the Networking and Kubernetes Code Examples repo](https://github.com/strongjz/Networking-and-Kubernetes) to follow code examples.

## Networking History

The purpose of a network is to exchange information from one system to another system.

A brief history of networking is following:

1. In **1969**, the Department of Defense sponsored the **Advanced Research Projects Agency Network (ARPANET)** was deployed at the UCLA, the Augementation Research Center at Stanford Research Institute, the UC Santa Barbara, and the University of Utah School of Computing.
1. In **1970**, communication between these nodes began using the **Network Control Protocol(NCP)**. NCP led to the development and use of the first computer-to-computer protocols like **Telnet** and **File Transfer Protocol(FTP)**.
1. In **1974**, [Vint Cerf](https://en.wikipedia.org/wiki/Vint_Cerf), [Yogen Dalal](https://en.wikipedia.org/wiki/List_of_Internet_pioneers#Yogen_Dalal), and [Carl Sunshine](https://en.wikipedia.org/wiki/List_of_Internet_pioneers#Carl_Sunshine) began drafting RFC 675 for **Transmission Control Protocol (TCP)**. TCP allowed for exchanging packets across different types of networks.
1. In **1981**, the **Internet Protocol (IP)**, defined in RFC 791, helped break out the responsibilities of TCP into a separate protocol, increasing the modularity of the network.
1. In **1983**, TCP/IP had become the only approved protocol on ARPANET, replacing the earlier NCP.
1. In **1991**, [Al Gore](https://en.wikipedia.org/wiki/Al_Gore) helped pass the National Information Infrastructure(NII) bill, inventing the internet. The Internet Engineering Task Force (IETF) was created accordingly. Nowadays standards for the internet are under the
management of the IETF. RFCs are published by the Internet Society and the IETF.

---
## OSI Model

![TCP/IP vs OSI](../../assets/img/kubernetes/networking/01-networking-introduction/osi-vs-tcpip.png)

| Layer number | Layer name | Protocol data unit | Function overview |
| :--- | :--- | :--- | :--- |
| 7 | Application | Data | provides the interface for applications like HTTP, DNS, and SSH. |
| 6 | Presentation | Data | character encoding, data compression, and encryption/decryption. |
| 5 | Session | Data | manages the connections between the local and remote applications. |
| 4 | Transport | Segment, datagram | provides **reliable data transfer services** to the upper layers through flow control, segmentation, and error control by TCP or UDP. |
| 3 | Network | Packet | transfer data flows from a host on **one network** to a host on **another network**. |
| 2 | Data Link | Frame | responsible for the host-to-host transfers on the same network. |
| 1 | Physical | Bit | sending and receiving of bitstreams over the medium. |

---
## TCP/IP

| Layer number | Layer name | Protocol data unit | Function overview |
| :--- | :--- | :--- | :--- |
| 5-7 | Application | Data | standardizes **process-to-process communication** by protocols like HTTPS |
| 4 | Transport | Segment, datagram | provides **reliable data transfer services** to the upper layers through flow control, segmentation, and error control by TCP or UDP. |
| 3 | Internet | Packet | responsible for transmitting data **between networks**. |
| 2 | Link | Frame | host-to-host transfers on the same network. Hosts are **identified by MAC addresses** on their network interface cards, and **determinted by** the host using **Address Resolution Protocol 9 (ARP)**. |
| 1 | Physical | Bit | details hardware standards such as IEEE 802.3. |

### Application

??? Warning "For the Apple Silicon mac users"

    [The official instruction to start up the Vagrant host](https://github.com/strongjz/Networking-and-Kubernetes/tree/master/chapter-1) is designed for the `x86` architecture. For the Apple Silicon mac users, please follow the below steps:

    1. Install `vagrant`, `qemu` and `vagrant-qemu` vagrant plugin:

        ``` bash
        # install vagrant
        brew tap hashicorp/tap
        brew install hashicorp/tap/hashicorp-vagrant

        # install qemu
        brew install qemu

        # install vagrant plugin
        vagrant plugin install vagrant-qemu
        ```

    2. Create your directory and initialize a known-good ARM64 box:

        ``` bash
        mkdir my-project && cd my-project
        vagrant init bento/ubuntu-22.04
        ```

    3. Open your `Vagrantfile` and replace the content with the following:

        ``` ruby
        Vagrant.configure("2") do |config|
          config.vm.box = "bento/ubuntu-22.04"

          # Use rsync to avoid the "SMB NT-compatible password" error
          config.vm.synced_folder ".", "/vagrant", type: "rsync"

          config.vm.provider "qemu" do |qe|
            qe.arch = "aarch64"
            qe.machine = "virt,accel=hvf" # Uses Apple's Hypervisor.framework for speed
            qe.cpu = "host"
            qe.net_device = "virtio-net-pci"
          end
        end
        ```

    4. Enter the VM:

        ``` bash
        vagrant up --provider qemu
        ```

#### HTTP

``` go
```


### Transport

### Network

### Internet Protocol

### Link Layer

### Revisiting Our Web Server

---
## Conclusion
