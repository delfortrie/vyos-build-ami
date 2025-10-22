# vi: set ft=ruby

$script = <<SCRIPT
#!/usr/bin/env bash


apt-get update
apt-get install -y python3-dev gcc python3-pip
apt-get install -y vim git ack-grep

pip3 install virtualenv

./bootstrap-virtualenv

echo "source env/bin/activate" > ~vagrant/.bash_profile
echo "cd /vagrant" >> ~vagrant/.bash_profile


SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "debian/trixie64"

  config.vm.provision :file, source: '~/.gitconfig', destination: '/home/vagrant/.gitconfig' if File.exist?(ENV['HOME'] + '/.gitconfig')
  config.vm.provision :file, source: '~/.vimrc', destination: '/home/vagrant/.vimrc' if File.exist?(ENV['HOME'] + '/.vimrc')
  config.vm.provision :file, source: '~/.vim', destination: '/home/vagrant/.vim' if File.exist?(ENV['HOME'] + '/.vim')
  config.vm.provision "shell", inline: $script
end

