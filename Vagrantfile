# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.no_install = true
  end

#  config.vm.provider "virtualbox" do |vb|
#    vb.cpus = 4
#    vb.memory = "8192"
#  end

  config.vm.network "forwarded_port", guest: 6443, host: 7443

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -s -
    sleep 10s
    # curl -sfL https://get.k3s.io | sudo sh - --write-kubeconfig-mode 644
    # sleep 10s
    k3s kubectl get node
    export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
    grep -q KUBECONFIG /home/vagrant/.profile || (echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> /home/vagrant/.profile)
    grep -q "alias k" /home/vagrant/.profile || (echo "alias k=\"k3s kubectl\"" >> /home/vagrant/.profile)
    kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
    kubectl patch storageclass local-path -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'    
    #download helm
    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh
    #Make instalation script executable
    chmod u+x install-helm.sh
    #Install helm
    ./install-helm.sh
    #Create tiller service account
    kubectl -n kube-system create serviceaccount tiller
    #Create cluster role binding for tiller
    kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
    kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
    # add incubator
    helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
    # install helm
    helm init --service-account tiller --kubeconfig /etc/rancher/k3s/k3s.yaml  
  SHELL
end