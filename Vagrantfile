Vagrant.configure("2") do |config|

  # If you run vagrant box add http://yum.oracle.com/boxes/oraclelinux/ol74/ol74.box then the box is local

  config.vm.box = "ol74"
  # config.vm.box_url = "http://yum.oracle.com/boxes/oraclelinux/ol74/ol74.box"

  # Provisioning: 
  # - Install Docker
  # - Create Btrfs partition on empty /dev/sdb included in ol74 box
  # - Configure Docker to use Btrfs for image and container management

  config.vm.provision "shell", inline: <<-'SHELL'

    # Fix the box to disable EPEL - kills yum 02/28/2018
    /bin/cp /vagrant/public-yum-ol7.repo /etc/yum.repos.d/

    # update the box
    yum update -y 
    
    * install nano
    yum install -y nano

    # install git
    yum install -y git

    # Install Docker
    yum install -y docker-engine btrfs-progs

    # Create a BTRFS partition
    mkfs.btrfs /dev/sdb

    # Configure docker to use our BTRFS volume
    eval $(blkid -o export /dev/sdb)
    cat <<-EOF >/etc/systemd/system/var-lib-docker.mount
	[Unit]
	Description = Docker Image Store
	After=network.target

	[Mount]
	What = UUID=${UUID}
	Where = /var/lib/docker
	Type = btrfs

	[Install]
	WantedBy = multi-user.target
EOF
    mkdir /var/lib/docker
    systemctl enable var-lib-docker.mount
    systemctl start var-lib-docker.mount
    cat <<-EOF >/etc/systemd/system/docker.service.d/var-lib-docker-mount.conf
	[Unit]
	Requires=var-lib-docker.mount
	After=var-lib-docker.mount
EOF

    # Ensure we use BTRFS driver
    sed -i "s/^DOCKER_STORAGE_OPTIONS=.*/DOCKER_STORAGE_OPTIONS='--storage-driver btrfs'/g" /etc/sysconfig/docker-storage

    # Add vagrant user to docker group
    usermod -a -G docker vagrant

    # Start Docker
    systemctl start docker
    systemctl enable docker

    echo "Your Docker VM is ready to use!"
    echo "Type vagrant ssh to get started."

    
  SHELL
end