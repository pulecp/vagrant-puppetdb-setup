
Vagrant.require_plugin 'vagrant-hostmanager'

domain       = 'dev.vstone.uni'
puppetmaster = "puppetmaster.#{domain}"


VIRTUAL_MACHINES = {
  :puppetmaster => {
    :ip             => '192.168.127.20',
    :hostname       => "puppetmaster01.#{domain}",
    :hostaliases    => [puppetmaster, "puppetdb.#{domain}"],
    :sourcedir      => 'puppet', ## Folder which contains the tree.
  },
  :client => {
    :puppet         => :client,
    :ip             => '192.168.127.50',
    :hostname       => "client.#{domain}",
    :forwards       => {
      80  => 40080,
      443 => 40443,
    },
  },
  :apply         => {
    :sourcedir      => 'puppet',
    :ip             => '192.168.127.60',
    :hostname       => "apply01.#{domain}",
    :forwards       => {
      80  => 46080,
      443 => 46443,
    },
  },
}

Vagrant.configure('2') do |config|
  ## Hostmanager configuration
  config.hostmanager.enabled = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  ## The base box we will use for ALl the hosts :)
  config.vm.box = "centos-6.x-64bit-puppet.3.x"
  config.vm.box_url = "http://packages.vstone.eu/vagrant-boxes/centos-6.x-64bit-latest.box"


  VIRTUAL_MACHINES.each do |name,cfg|
    config.vm.define name do |vm_config|

      environment = cfg[:environment] || 'develop'
      sourcedir   = cfg[:sourcedir] || 'puppet'
      graphdir    = "/vagrant/graphs/#{name}"

      ## Configure basics.
      vm_config.vm.box                = cfg[:box]           if cfg[:box]
      vm_config.vm.box_url            = cfg[:box_url]       if cfg[:box_url]
      vm_config.vm.hostname           = cfg[:hostname]      if cfg[:hostname]
      vm_config.hostmanager.aliases   = cfg[:hostaliases]   if cfg[:hostaliases]
      vm_config.vm.network :private_network, ip: cfg[:ip]   if cfg[:ip]

      if cfg[:forwards]
        cfg[:forwards].each do |guest, host|
          vm_config.vm.network :forwarded_port, guest: guest, host: host
        end
      end

      # Creates the directory for puppet runs using --graph.
      unless Dir.exists?(File.expand_path(File.join(File.dirname(__FILE__), "./graphs/#{name}")))
        Dir.mkdir(File.expand_path(File.join(File.dirname(__FILE__), "./graphs/#{name}")))
      end

      ## Update hosts file on the machine.
      vm_config.vm.provision :hostmanager

      ## Run scripts that match pre-<nameofthebox>.sh
      if File.exists?(File.expand_path(File.join(File.dirname(__FILE__), "./scripts/pre-#{name}.sh")))
        vm_config.vm.provision :shell do |shell|
          shell.path = File.expand_path(File.join(File.dirname(__FILE__), "./scripts/pre-#{name}.sh"))
          shell.args = "#{environment} #{sourcedir}"
        end
      end

      ## If :puppet => :client, do a puppet run against the master
      if cfg[:puppet] and cfg[:puppet] == :client
        vm_config.vm.provision :puppet_server do |puppet|
          puppet.puppet_server = puppetmaster
          puppet.options = "--verbose --debug --environment #{environment} --test --trace --graph --graphdir #{graphdir}"
        end
      else
        ## Run the shell script that matches the name of the box or the default script.
        vm_config.vm.provision :shell do |shell|
          if File.exists?(File.expand_path(File.join(File.dirname(__FILE__), "./scripts/puppetrun-#{name}.sh")))
            shell.path = "scripts/puppetrun-#{name}.sh"
          else
            shell.path = "scripts/puppetrun.sh"
          end
          shell.args = "#{environment} #{name} #{sourcedir}"
        end
      end
    end
  end
end


