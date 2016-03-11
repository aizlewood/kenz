require 'yaml'

Vagrant.configure("2") do |config|

    settings = YAML::load(File.read(File.expand_path("../storage/config/dump.yml", __FILE__)))

    ### Box settings #####################################################################

    box = settings['devbox']['boxes'][settings['devbox']['box']]
    config.vm.box = box['name']
    config.vm.box_url = box['url'].strip
    config.vm.hostname = "devbox.clearleft.host"
    config.ssh.forward_agent = true;

    ### Set up networking ################################################################

    config.vm.network :private_network, ip: settings['devbox']["ip"]
    config.vm.network "forwarded_port", guest: 22, host: 2202
    config.vm.network "forwarded_port", guest: 443, host: 4430
    config.vm.network "forwarded_port", guest: 3306, host: 33060
    config.vm.network "forwarded_port", guest: 5432, host: 54320

    ### Virtual machine settings #########################################################

    config.vm.provider "virtualbox" do |vb|
      vb.name = 'devbox'
      vb.customize ["modifyvm", :id, "--memory", settings["memory"] ||= "2048"]
      vb.customize ["modifyvm", :id, "--cpus", settings["cpus"] ||= "2"]
      vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
    end

    ### SSH ###############################################################################

    config.vm.provision "shell" do |s|
        s.inline = "cat /vagrant/vendor/clearleft/elf-core/resources/keys/id_rsa.devbox.pub >> ~/.ssh/authorized_keys"
    end

    ### Shared folders ##########################################################

    devboxRemote = settings['remotes']['servers']['devbox']
    config.vm.synced_folder settings['devbox']['host_sites_dir'], devboxRemote['sites_dir'], :nfs => true, :mount_options =>  ['actimeo=1', 'rw', 'vers=3', 'udp', 'fsc', 'nolock', 'noatime']

    ### Git config ########################################################################

    if settings.has_key?("git")
      config.vm.provision "shell" do |s|
            s.privileged = false
            s.inline = "git config --global user.name \"$1\" && git config --global user.email \"$2\""
            s.args = [settings["git"]["name"], settings["git"]["email"]]
        end
    end

    ### Updates ###########################################################################

    config.vm.provision "shell" do |s|
      s.inline = "composer self-update"
    end

end
