# frozen_string_literal: true

# -*- mode: ruby -*-
# vi: set ft=ruby :
# SPDX-license-identifier: Apache-2.0
##############################################################################
# Copyright (c) 2025
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

num_nodes = ENV['NUM_NODES'] || '2'

Vagrant.configure('2') do |config|
  config.vm.provider :libvirt
  config.vm.provider :virtualbox

  config.vm.box = 'generic/ubuntu2204'
  config.vm.box_version = '4.3.12'
  config.vm.synced_folder './', '/vagrant'

  config.vm.provider :libvirt do |v, override|
    override.vm.synced_folder './', '/vagrant', type: 'nfs', nfs_version: ENV.fetch('VAGRANT_NFS_VERSION', 3)
    v.cpu_mode = 'host-passthrough'
    v.random_hostname = true
    v.management_network_address = '10.0.2.0/24'
    v.management_network_name = 'administration'
    v.management_network_mode = 'nat'
    v.cpu_mode = 'host-passthrough'
  end
  config.vm.provider 'virtualbox' do |v|
    v.gui = false
    v.customize ['modifyvm', :id, '--nictype1', 'virtio', '--cableconnected1', 'on']
    # Enable nested paging for memory management in hardware
    v.customize ['modifyvm', :id, '--nestedpaging', 'on']
    # Use large pages to reduce Translation Lookaside Buffers usage
    v.customize ['modifyvm', :id, '--largepages', 'on']
    # Use virtual processor identifiers  to accelerate context switching
    v.customize ['modifyvm', :id, '--vtxvpid', 'on']
  end

  (1..num_nodes.to_i).each do |machine_id|
    config.vm.define "node#{machine_id.to_s.rjust(2, '0')}" do |machine|
      machine.vm.hostname = "node#{machine_id.to_s.rjust(2, '0')}"

      %i[virtualbox libvirt].each do |provider|
        machine.vm.provider provider do |p|
          p.memory = ENV['MEMORY'] || (4 * 1024)
        end
      end

      config.vm.provider :virtualbox do |v|
        v.customize ['modifyvm', :id, '--nested-hw-virt', 'on']
      end

      machine.vm.provider :libvirt do |v|
        v.nested = true
      end
      machine.vm.provision :ansible_local do |ansible|
        ansible.playbook = 'playbook.yml'
      end
    end
  end

  if !ENV['http_proxy'].nil? && !ENV['https_proxy'].nil? && Vagrant.has_plugin?('vagrant-proxyconf')
    config.proxy.http = ENV['http_proxy'] || ENV['HTTP_PROXY'] || ''
    config.proxy.https    = ENV['https_proxy'] || ENV['HTTPS_PROXY'] || ''
    config.proxy.no_proxy = no_proxy
    config.proxy.enabled = { docker: false }
  end
end
