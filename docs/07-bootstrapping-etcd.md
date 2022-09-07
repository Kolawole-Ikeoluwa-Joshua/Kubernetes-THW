# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

### Prerequisites

The commands in this lab must be run on each controller instance: `master-1`, and `master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time.

Quick start tips:
* Install tmux on master-1 node
```
sudo apt install tmux
```
* Start a new session
```
tmux new
```
* Split window into horizontal planes
```
press & release [ctrl + b], followed by [%]
``` 
* Switch to second pane and SSH into master-2 node
```
press & release [ctrl + b], followed by  ‘{‘ or ‘}’ (to switch panes)

run [ssh master-2] to SSH into master-2 node
```
* Synchronize both panes
```
press & release [ctrl + b], followed by [:]

enter the command [setw synchronize-panes]

enter the command [setw synchronize-panes] again to deactivate sync
``` 

![syncedtmux](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/sync%20tmux%20panes.png)

* To exit tmux session run the command in any pane:
```
tmux kill-session
```
