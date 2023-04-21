Vagrant.configure("2") do |config|
  config.vm.box = "almalinux/9"
  config.vm.network "forwarded_port", guest: 8980, host: 8980, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"

  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = 3072
    v.vmx["numvcpus"] = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    dnf -y install vim-enhanced nmap-ncat net-snmp net-snmp-utils rrdtool
    sed -i -e '/^view.*systemview.*included.*\.1\.3\.6\.1\.2\.1\.1/s/\.1\.3\.6\.1\.2\.1\.1$/.1/' /etc/snmp/snmpd.conf
    systemctl enable snmpd
    systemctl start snmpd
    dnf -y install java-11-openjdk-devel
    dnf install -y postgresql-server postgresql
    postgresql-setup initdb
    sed -i -e '/^host.*all.*all.*ident$/s/ident/trust/' /var/lib/pgsql/data/pg_hba.conf
    systemctl enable --now postgresql
    curl -1sLf 'https://packages.opennms.com/public/common/setup.rpm.sh' | sudo -E bash
    curl -1sLf 'https://packages.opennms.com/public/stable/setup.rpm.sh' | sudo -E bash
    dnf -y install jrrd2 iplike-pgsql13
    dnf -y install haveged opennms-core opennms-webapp-jetty grafana opennms-helm
    hostnamectl set-hostname horizon-$(rpm -q opennms-core | awk -F- '{ print $3 }' | sed -e 's/\\./-/g')
    /opt/opennms/bin/runjava -s
    /opt/opennms/bin/install -dis
    /sbin/install_iplike-13.sh
    systemctl enable --now haveged
    systemctl enable --now opennms
    systemctl enable --now grafana-server
    sed -i -e '/^Defaults.*secure_path = /s,$,:/opt/opennms/bin,' /etc/sudoers
    #/usr/bin/install -o vagrant -g vagrant -m 0600 /vagrant/dot_bash_history /home/vagrant/.bash_history
  SHELL
end
