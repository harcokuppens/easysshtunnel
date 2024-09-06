# Easy ssh tunnel

Easily create a ssh tunnel or bridge.

<!--ts-->
   * [Description](#description)
   * [Examples using sshtunnel and sshbridge](#examples-using-sshtunnel-and-sshbridge)
   * [Installation](#installation)
   * [Help documentation for sshtunnel and sshbridge](#help-documentation-for-sshtunnel-and-sshbridge)
      * [sshtunnel](#sshtunnel)
      * [sshbridge](#sshbridge)

<!-- Created by https://github.com/ekalinin/github-markdown-toc -->
<!-- Added by: harcok, at: do sep  5 08:51:13 CEST 2024 -->

<!--te-->

## Description

The `ssh` command is a great tool to make SSH tunnels for either

- **tunneling**: making an end-to-end encrypted tunnel to protect data traffic which is send through
  it from eavesdropping
- **bridging**: setup an SSH connection to a bridge SSH server to bridge data traffic over a
  firewall to a server behind it

However the syntax for the `ssh` command to implement above cases is a bit tricky in details.
Everytime I want to setup such connection I have to figure out the details again, which costs me a
lot of time everytime.

Therefore I decided to make 2 simple wrapper commands over the `ssh` command to make it more easy
and intuitive to create a new SSH tunnel or bridge within a few seconds:

1. [sshtunnel](#sshtunnel) - create an end-to-end encrypted SSH tunnel from a local port on
   localhost to a local port on a SSH server
2. [sshbridge](#sshbridge) - use SSH to bridge TCP traffic over a firewall.

These commands are more intuitive because their arguments specify the linear order in which the data
is flowing. So by just thinking how you want the traffic to go, you can just immediately write out
the command.

## Examples using sshtunnel and sshbridge

**Tunneling** VNC traffic to an VNC server behind a firewall with the help of ssh server open to the
outside world 'eg. lilo.science.ru.nl' where the VNCSERVER itself also runs a SSH server so we can
send all VNC message safely over an end-to-end encrypted tunnel. Note that the VNC protocol does not
encrypt its traffic so we must supply an end-to-end encrypted tunnel for it to prevent
eavesdropping.

    # make an encrypted ssh tunnel to VNCSERVER to prevent eavesdropping
    sshtunnel 5900 VNCSERVER
    # executes: ssh -N -L 5900:localhost:5900 VNCSERVER

    # also bridge the tunnel over a firewall using lilo.science.ru.nl
    sshtunnel 5900  lilo.science.ru.nl VNCSERVER
    # executes: ssh -N -J lilo.science.ru.nl -L 5900:localhost:5900 VNCSERVER

    # use locally another port if the local machine itself is running a VNC server
    sshtunnel 15900  lilo.science.ru.nl VNCSERVER 5900
    # executes: ssh -N -J lilo.science.ru.nl -L 15900:localhost:5900 VNCSERVER

**Bridging** RDP traffic to a RDP server behind a firewall with the help of a SSH bridge server open
to the outside world 'eg. lilo.science.ru.nl' Note that the RDP protocol supports encryption by
itself, so only passing the firewall using an SSH bridge server is needed.

    # bridge local port 3389 via lilo.science.ru.nl bridge to RDPSERVER (on same port)
    sshbridge 3389 lilo.science.ru.nl RDPSERVER
    # executes: ssh -N -L 3389:RDPSERVER:3389 lilo.science.ru.nl
    # bridge local port 13389 via lilo.science.ru.nl bridge to port 3389 on the RDPSERVER
    sshbridge 13389 lilo.science.ru.nl RDPSERVER 3389
    # executes: ssh -N -L 13389:RDPSERVER:3389 lilo.science.ru.nl

Note, if you want guaranteed end-to-end encryption then you can just change the command 'sshbridge'
to 'sshtunnel', but you must enable a SSH server on the endpoint server 'RDPSERVER', otherwise the
SSH tunnel cannot be made all the way:

    sshtunnel 13389 lilo.science.ru.nl RDPSERVER 3389
    # RDPSERVER runs also an SSH server

## Installation

Just execute the following steps to install [sshtunnel](#sshtunnel) and [sshbridge](#sshbridge) into
`/usr/local/bin/`:

    INSTALL_DIR=/usr/local/bin  # make sure INSTALL_DIR is in your PATH environment variable
    DOWNLOAD_URL="https://raw.githubusercontent.com/harcokuppens/easysshtunnel/main/bin/"
    sudo curl -Lo "${INSTALL_DIR}/sshtunnel"  "$DOWNLOAD_URL/sshtunnel"
    sudo curl -Lo "${INSTALL_DIR}/sshbridge"  "$DOWNLOAD_URL/sshbridge"
    sudo chmod a+x "${INSTALL_DIR}/sshtunnel" "${INSTALL_DIR}/sshtunnel"
    sudo chmod a+x "${INSTALL_DIR}/sshtunnel" "${INSTALL_DIR}/sshbridge"

On a Windows machine you can install [Git for Windows](https://gitforwindows.org) which provides you
with a terminal running bash on which you can run ssh commands, and above installation instructions
to install sshtunnel and sshbridge.

## Help documentation for sshtunnel and sshbridge

### sshtunnel

    $ sshtunnel -h

    NAME
      sshtunnel - create an end-to-end encrypted SSH tunnel from a local port on localhost to a local port on a SSH server

    USAGE
      sshtunnel LOCALPORT  [SSHHOST_1 ... SSHHOST_N-1] SSHHOST_N [REMOTEPORT]

    WHERE
      Each SSHHOST_X can be specified as: [USER_X@]SSHSERVER_X[:SSHPORT_X]
      If REMOTEPORT is not given, then REMOTEPORT=LOCALPORT.

      For more information run: sshtunnel -h

    DESCRIPTION
      Sshtunnel creates an end-to-end encrypted SSH tunnel from localhost to a remote SSH server.
      The SSH tunnel forwards traffic coming into the local port LOCALPORT on localhost, safely encrypted,
      to a port REMOTEPORT on the localhost interface on the remote SSH server SSHHOST_N.
      We can have an additional SSH host(s) to also bridge traffic over firewall(s).

      The encrypted SSH-tunnel is created to SSHHOST_1, which extends the SSHtunnel to SSHHOST_2, ... to SSHHOST_N.
      From the last SSHHOST_N all traffic is forwarded locally to localhost:REMOTEPORT.
      That is on the last SSHHOST_N all traffic is sent to port REMOTEPORT on the internal localhost interface.
      An end-to-end encrypted SSH-tunnel is made to the remote service running on internal port REMOTEPORT on SSHHOST_N.

      It executes the following ssh command:

           ssh -N  [-J  SSHHOST_1,SSHHOST_2,...,SSHHOST_N-1]  -L LOCALPORT:localhost:REMOTEPORT SSHHOST_N

      So the destination machine SSHHOST_N must run a SSH server so that you can make an end-to-end encrypted tunnel.
      For the service running on the internal port REMOTEPORT on SSHHOST_N the connection is coming from the localhost interface,
      which allows us to connect to services which are not open to the outside world!

    EXAMPLE
      Tunneling VNC traffic to an VNC server behind a firewall with
      the help of ssh server open to the outside world 'eg. lilo.science.ru.nl'
      where the VNCSERVER itself also runs a SSH server so we can
      send all VNC message safely over an end-to-end encrypted tunnel.
      Note that the VNC protocol does not encrypt its traffic so we must supply
      an end-to-end encrypted tunnel for it to prevent eavesdropping.

        # make an encrypted ssh tunnel to VNCSERVER to prevent eavesdropping
        sshtunnel 5900 VNCSERVER
        # executes: ssh -N -L 5900:localhost:5900 VNCSERVER

        # also bridge the tunnel over a firewall using lilo.science.ru.nl
        sshtunnel 5900  lilo.science.ru.nl VNCSERVER
        # executes: ssh -N -J lilo.science.ru.nl -L 5900:localhost:5900 VNCSERVER

        # use locally another port if the local machine itself is running a VNC server
        sshtunnel 15900  lilo.science.ru.nl VNCSERVER 5900
        # executes: ssh -N -J lilo.science.ru.nl -L 15900:localhost:5900 VNCSERVER

### sshbridge

    $ sshbridge -h

    NAME
      sshbridge - Use SSH to bridge TCP traffic over a firewall.

    USAGE
      sshbridge LOCALPORT  [SSHHOST_1 ... SSHHOST_N-2] SSHHOST_N REMOTESERVER [REMOTEPORT]

    WHERE
      Each SSHHOST_X can be specified as: [USER_X@]SSHSERVER_X[:SSHPORT_X]
      If REMOTEPORT is not given, then REMOTEPORT=LOCALPORT.

    DESCRIPTION
      Use SSH to bridge TCP traffic over a firewall.
      We can have multiple SSH hosts to do multiple firewall bridging.
      Although normally you would need only one.

      The encrypted SSH-tunnel is created to SSHHOST_1, which extends the SSHtunnel to SSHHOST_2, ... to SSHHOST_N.
      From the last SSHHOST_N all traffic is forwarded over a none-encrypted connection to REMOTEHOST:REMOTEPORT

      It executes the following ssh command:

             ssh -N  [-J SSHHOST_1,SSHHOST_2,...SSHHOST_N-1]  -L LOCALPORT:REMOTESERVER:REMOTEPORT SSHHOST_N

      All traffic comes from the outside world into REMOTESERVER on port REMOTEPORT
      where REMOTESERVER does not have to run a SSH server.

    EXAMPLE
      Bridge RDP traffic to an RDP server behind a firewall with
      the help of a SSH bridge server open to the outside world 'eg. lilo.science.ru.nl'
      Note that the RDP protocol supports encryption by itself, so only
      passing the firewall using an SSH bridge server is needed.

        # bridge local port 3389 via lilo.science.ru.nl bridge to RDPSERVER (on same port)
        sshbridge 3389 lilo.science.ru.nl RDPSERVER
        # executes: ssh -N -L 3389:RDPSERVER:3389 lilo.science.ru.nl
        # bridge local port 13389 via lilo.science.ru.nl bridge to port 3389 on the RDPSERVER
        sshbridge 13389 lilo.science.ru.nl RDPSERVER 3389
        # executes: ssh -N -L 13389:RDPSERVER:3389 lilo.science.ru.nl

      Note, if you want guaranteed end-to-end encryption then you can just change
      the command  'sshbridge' to 'sshtunnel', but you must enable a SSH
      server on the endpoint server 'RDPSERVER', otherwise the SSH tunnel
      cannot be made all the way:

        sshtunnel 13389 lilo.science.ru.nl RDPSERVER 3389
        # RDPSERVER runs also an SSH server
