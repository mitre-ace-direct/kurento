Initial README
=======
# Installation of Kurento on Ubuntu Server 18.04 LTS on AWS EC2
---
### Introduction
In ACE Direct v4.0, two major components are required on the  Kurento hosting instance - Kurento Media Server and Signaling Server.

---
### Part I - Installation of *kurento-media-server*

    ## Update *apt* and install *kurento*
    sudo apt-get update

    ## Import the key from Kurento TEAM
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5AFA7A83
    ## In certain cases where import with above command may fail, try using hkp://keyserver.ubuntu.com:80 and make sure proxy is set via --keyserver-options if there is one.
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 5AFA7A83


    ## Set DISTRO to "bionic" for Ubuntu 18.04, "xenial" for Ubuntu 16.04
    DISTRO="bionic"

    ## Adding repo to source
    sudo tee "/etc/apt/sources.list.d/kurento.list" >/dev/null << EOF
    # Kurento Media Server - Release packages
    deb [arch=amd64] http://ubuntu.openvidu.io/6.11.0 $DISTRO kms6
    EOF
    unset DISTRO

    ## Install kurento-media-server
    sudo apt-get update && sudo apt-get install --yes kurento-media-server


### Part II - Combo Certificate
Kurento uses a cerficate file that combines private key, certificate and intermediate CA.

    ## Create a combo certificate for Kurento
    cat key.pem fullchain.pem > server.pem

### Part III - Modify configuration files under `/etc/kurento`

**_/etc/kurento/kurento.conf.json_**: The main configuration file to provide settings of Kurento Media Server. Defines the WebSocket(WS)/WebSocket Secure(WSS) ports and path to combo certificates. See https://doc-kurento.readthedocs.io/en/6.9.0/features/security.html#configure-kurento-media-server-to-use-secure-websocket-wss for more details. <sup>[1](#fn1)</sup>

**_/etc/kurento/modules/kurento/WebRtcEndpoint.conf.ini_**: Specific parameters for WebRtcEndpoint.<sup>[1](#fn1)</sup>


**_/etc/kurento/modules/kurento/MediaElement.conf.ini_**: Generic parameters for all kinds of MediaElement.<sup>[2](#fn2)</sup>

**_/etc/kurento/modules/kurento/SdpEndpoint.conf.ini_**: Audio/video parameters for SdpEndpoints (i.e. WebRtcEndpoint and RtpEndpoint).<sup>[2](#fn2)</sup>

**_/etc/kurento/modules/kurento/HttpEndpoint.conf.ini_**: Specific parameters for HttpEndpoint.<sup>[2](#fn2)</sup>


---
<a name="fn1"><sup>[1]</sup></a> Check repo directory in **acedirect-kurento/confs/kurento/** for the examples  </br>
<a name="fn2"><sup>[2]</sup></a> Check the [Kurento Github](https://github.com/Kurento/kms-core/tree/6.11.0/src/server/config) page

### Part III - Runtime Configuration

#### Set up privacy video for ACE Direct

1. Log into the Kurento media server.
1. `sftp` the `videoPrivacy.webm` file to the Kurento media server.
1. From a terminal:

        ```bash
        $  sudo cp videoPrivacy.webm /tmp/media/.
        $  sudo chown -R kurento:kurento /tmp/media
        $  sudo chmod 644 /tmp/media/videoPrivacy.webm
        $
        ```

1. From the ACE Direct node server, edit `dat/config.json` to have this entry:

        ```bash
        "media_server": {
            "privacy_video_url": "file:///tmp/media/videoPrivacy.webm"
        },
        ```
