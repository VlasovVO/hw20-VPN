# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :tap => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.10', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "vpn-net"},
                ]
  },

  :taptun => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.20', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "vpn-net"},
                   {ip: '192.168.0.130', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "vpn-net"},
        ]
  },
  :tun => {
    :box_name => "centos/7",
    :net => [
      {ip: '192.168.0.140', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "vpn-net"},
    ]
  },
    :openvpn => {
      :box_name => "centos/7",
      :net => [
        {ip: '192.168.1.40', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "vpn-net"},
      ],
      :private_net => {:ip => '192.168.1.40'},

}

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s

      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", ipconf
      end

      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end

      if boxconfig.key?(:private_net)
        box.vm.network "private_network", ip: boxconfig[:private_net][:ip]
      end

      box.vm.provision "shell", inline: <<-SHELL
                mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL

      case boxname.to_s
      when "tap"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            ip link add tap0 type gretap local 192.168.0.10 remote 192.168.0.20
            ip link set up tap0
            ip a a 192.168.1.1/30 dev tap0
            SHELL
      when "taptun"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            ip link add tap0 type gretap local 192.168.0.20 remote 192.168.0.10
            ip link set up tap0
            ip a a 192.168.1.2/30 dev tap0
            ip tunnel add tun0 mode gre local 192.168.0.130 remote 192.168.0.140
            ip link set up tun0
            ip a a dev tun0 192.168.2.1 peer 192.168.2.2/32
            SHELL
      when "tun"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            ip tunnel add tun0 mode gre local 192.168.0.140 remote 192.168.0.130
            ip link set up tun0
            ip a a dev tun0 192.168.2.2 peer 192.168.2.1/32
            SHELL

      when "openvpn"
        config.vm.network "forwarded_port", guest: 8080, host: 8080
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum -y install epel-release
            yum -y install openvpn

            cp /vagrant/openvpn-server-ca.crt /etc/openvpn/ca.crt
            cp /vagrant/openvpn-server.key /etc/openvpn/server.key
            cp /vagrant/openvpn-server.crt /etc/openvpn/server.crt
            cp /vagrant/openvpn-server-dh.pem /etc/openvpn/dh2048.pem
            cp /vagrant/server.conf /etc/openvpn/server.conf

            systemctl enable openvpn@server.service
            systemctl start openvpn@server.service

            SHELL
      end
    end
  end
end
