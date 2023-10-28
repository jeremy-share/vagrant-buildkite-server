# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'environment.rb'
include Secrets

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "90"]
  end

  config.vm.box = "ubuntu/jammy64"  # Ubuntu 22.04 LTS

  buildkite_token = Secrets::BUILDKITE_AGENT_TOKEN
  unless buildkite_token
    raise "ERROR: BUILDKITE_AGENT_TOKEN is not set in the Secrets module!"
  end

  config.vm.provision "shell", privileged: true, env: {"BUILDKITE_AGENT_TOKEN" => buildkite_token}, inline: <<-SHELL
    # Ensure we have the required tools for adding repositories
    sudo apt-get update
    sudo apt-get install -y software-properties-common

    # Add the Buildkite Agent repository to the sources.list.d directory
    echo deb https://apt.buildkite.com/buildkite-agent stable main | sudo tee /etc/apt/sources.list.d/buildkite-agent.list

    # Add the Buildkite Agent’s GPG public key
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 32A37959C2FA5C3C99EFBC32A79206696452D198

    # Install Buildkite Agent
    sudo apt-get update
    sudo apt-get install -y buildkite-agent

    # Configure Buildkite Agent with token
    sudo sed -i "s/xxx/$BUILDKITE_AGENT_TOKEN/g" /etc/buildkite-agent/buildkite-agent.cfg

    # Start and enable Buildkite Agent
    sudo systemctl enable buildkite-agent
    sudo systemctl start buildkite-agent
  SHELL
end
