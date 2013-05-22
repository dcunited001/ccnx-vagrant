# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'erubis'

def readconf(filename)
  #reads a yaml file in 'config/vagrant'
  pwd = File.dirname(__FILE__)
  conf_file = File.join(pwd, "config", filename)
  vconf = Erubis::Eruby.new(File.open(conf_file, File::RDONLY).read)
  vconf = vconf.result(def_host_share: pwd)
  vconf = YAML::load(vconf)
end

Vagrant::Config.run do |config|
  # may be possible to automagically configure with hash
  @vconf = readconf("vagrant.yml")['defaults']
  @vbox_customize = ["modifyvm", :id, "--memory", "2048"]

  config.vm.define :chef do |conf|

    #basics:
    conf.vm.box = @vconf['box']
    conf.vm.host_name = @vconf['host_name']
    conf.vm.boot_mode = @vconf['boot_mode'].to_sym
    conf.vm.customize @vbox_customize

    #network:
    conf.vm.network @vconf['net']['mode'].to_sym,
      @vconf['net']['ip'],
      netmask: @vconf['net']['host']

    #ports:
    conf.vm.forward_port 80, @vconf['port']['http']
    conf.vm.forward_port 22, @vconf['port']['ssh'], auto: true

    #ssh:
    conf.ssh.username = @vconf['ssh']['username']
    conf.ssh.shell = @vconf['ssh']['shell']

    #fileshare:
    shopts = {}.merge({nfs: (@vconf['share']['type'] == 'nfs') })
    shargs = ['devshare',
                 @vconf['share']['guestpath'],
                 @vconf['share']['hostpath'],
                 shopts]
    conf.vm.share_folder(*shargs)

  end
end
