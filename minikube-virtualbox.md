# Configuring minikube with virtualbox driver
by: [@kauanpecanha](https://github.com/kauanpecanha)

Description: These instructions are intended to teach the process to install minikube and configure a virtualbox cluster (in a way that it works, once that the official documentation might not work well).

## Specs
OS: fedora 43
Kubernetes: minikube
driver: virtualbox

## Instructions
Warning: these instructions might be outdated. be aware with the specific version used.

Download the latest release with the command:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install kubectl
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Enable RPM Fusion
```bash
sudo dnf install -y \
https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-43.noarch.rpm \
https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-43.noarch.rpm
```

Install virtualbox
```bash
sudo dnf install -y akmod-VirtualBox VirtualBox
sudo akmods --kernels 6.19.7-200.fc43.x86_64 && systemctl restart vboxdrv.service
```

Setup Virtualbox Minikube clustr
```bash
minikube start --driver=virtualbox -p <name>
```

## Reference
- [minikube virtualbox documentation](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/)

Made with ❤️ in 🇧🇷