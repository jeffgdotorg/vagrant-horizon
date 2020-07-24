Vagrant.configure("2") do |config|
  config.vm.box = "centos/8"
  config.vm.network "forwarded_port", guest: 8980, host: 8980, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    yum -y install vim-enhanced nmap-ncat net-snmp net-snmp-utils rrdtool
    sed -i -e '/^view.*systemview.*included.*\.1\.3\.6\.1\.2\.1\.1/s/\.1\.3\.6\.1\.2\.1\.1$/.1/' /etc/snmp/snmpd.conf
    systemctl enable snmpd
    systemctl start snmpd
    yum -y install epel-release
    yum -y install haveged
    yum -y install java-11-openjdk-devel
    yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    yum --disablerepo=AppStream -y install postgresql12-server postgresql12
    systemctl enable haveged
    systemctl start haveged
    /usr/pgsql-12/bin/postgresql-12-setup initdb
    sed -i -e '/^host.*all.*all.*ident$/s/ident/trust/' /var/lib/pgsql/12/data/pg_hba.conf
    systemctl enable postgresql-12
    systemctl start postgresql-12
    yum -y install http://yum.opennms.org/repofiles/opennms-repo-stable-rhel8.noarch.rpm
    yum -y install jrrd2 iplike-pgsql12
    yum -y install opennms-core opennms-webapp-jetty grafana opennms-helm
    hostnamectl set-hostname horizon-$(rpm -q opennms-core | awk -F- '{ print $3 }' | sed -e 's/\\./-/g')
    /opt/opennms/bin/runjava -s
    /opt/opennms/bin/install -dis
    /sbin/install_iplike-12.sh
    systemctl enable opennms
    systemctl start opennms
    systemctl enable grafana-server
    systemctl start grafana-server
    sed -i -e '/^Defaults.*secure_path = /s,$,:/opt/opennms/bin,' /etc/sudoers
    /usr/bin/install -o vagrant -g vagrant -m 0600 /vagrant/dot_bash_history /home/vagrant/.bash_history
  SHELL
end
