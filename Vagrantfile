# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

vagrant_config = YAML.load_file('./config.yaml')
options = {
  PROVIDER: vagrant_config['provider'].to_sym,
  CLIENTS:  vagrant_config['clients'],
  FORCE:    vagrant_config['force'],
  SSH_KEY:  vagrant_config['ssh_private_key']
}

def digital_ocean(config, options)
  config.vm.provider :digital_ocean do |provider, override|
    override.ssh.private_key_path = options[:SSH_KEY]
    override.vm.box               = 'digital_ocean'
    override.vm.box_url           = 'https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box'

    provider.image        = 'ubuntu-16-04-x64'
    provider.name         = 'openvpn'
    provider.region       = options[:REGION]
    provider.ssh_key_name = options[:SSH_KEY_NAME]
    provider.size         = options[:SIZE]
    provider.token        = options[:TOKEN]
  end
end

VAGRANTFILE_API_VERSION = '2'.freeze
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define 'openvpn' do |name|
    name.vm.hostname = 'openvpn'
  end

  case options[:PROVIDER]
  when :digital_ocean
    do_options = options.merge(
      REGION:       vagrant_config['do_region'],
      SIZE:         vagrant_config['do_size'],
      SSH_KEY_NAME: vagrant_config['do_ssh_key_name'],
      TOKEN:        vagrant_config['do_token'] ? vagrant_config['do_token'] : File.read(File.expand_path(vagrant_config['do_token_file'])).chomp
    )
    digital_ocean(config, do_options)
  else
    abort('Unsupported provider')
  end

  config.vm.synced_folder '.', '/vagrant',
                          type: 'rsync', rsync__exclude: '.git/',
                          rsync__args: ['-v', '-a', '-z', '-L']

  config.vm.provision 'shell', path: 'provisioning/1.prep.sh'
  config.vm.provision 'shell', path: 'provisioning/2.forwarding.sh'
  config.vm.provision 'shell', path: 'provisioning/3.firewall.sh'
  config.vm.provision 'shell', path: 'provisioning/4.server.sh'

  args = options[:FORCE] ? ["-c #{options[:CLIENTS]}", '-f'] : ["-c #{options[:CLIENTS]}"]
  config.vm.provision 'shell', path: 'provisioning/5.client.sh', args: args
end
