## Write an app

Here the application is just a `GET /` server that says hello.

Bundler is configured to user `./vendor/bundler` as the directory
to store all the dependencies of the project. It'll allow us to
deploy those dependencies as the rest of the application's code.

To runde the application locally, run the following commands:

~~~ bash
    bundle install
    bundle exec puma
~~~

I assume that you have Ruby and the bundler gem installed.

## Spawn a VM

1. Create a box.
2. Install the latest Ubuntu Server 16.04 LTS.
3. Select the `openssh-server` during the installation process.

In order to acces the VM from the hosts, I used port rediection
at the VM level. I redirected port 3022 of the host to the port
22 of the guest.

To do so, open the machine _settings_ menu, go in the _network_ panel,
display the _advanced_ options of the adapter, and then there is the
_port forwarding_ button.

## Get SSH working

I used the user `app` when I installed the VM. I want to allow
connections via SSH only. To do so, I start by adding my public ssh
key to the `authorized_keys` of my `app` user.

1. Login with `app` on the VM
2. Fill the `.ssh/authorized_keys` with appropriate data:
~~~ bash
    mkdir .ssh
    wget https://github.com/nicoolas25.keys
    mv nicoolas25.keys .ssh/authorized_keys
~~~
3. Test the connection from the host:

~~~ bash
    ssh -p 3022 app@127.0.0.1
~~~

## Install Ruby

To execute a Ruby program, we'll need a Ruby interpreter. Install it
on the VM from the default repositories:

~~~ bash
    sudo apt-get install ruby2.3
    sudo gem install bundler
~~~

## Prepare the app folder

Still on the VM, lets create a folder where the Ruby application will live:

~~~ bash
    mkdir ~/app
~~~

Once we have this place on the VM, we can deploy the necessary files from the
host to the VM. The same would be done from a dev box to a server box.

~~~ bash
    rsync -av -e 'ssh -p 3022' ./ app@127.0.0.1:~/app/
~~~

## Start the application

Start manually the application with the following commands:

~~~ bash
    cd ~/app
    RACK_ENV=production bundle exec puma
~~~

## Conclusion

Congrats you just ran a Ruby application on a remote server!

Here are the next steps:

* Add a `systemd` service config that ensure that puma is always running.
* Create a rollback system using symlinks and directories.
* Automate.
