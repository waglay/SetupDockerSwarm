# SetupDockerSwarm
## Prerequisites
---
Well this is not really important but I love going through the base of everything which led me ko go through the overlay concept before starting docker swarm. There is a underlay overlay concept where as long as the devices are physically connected, request can be send from one end to other with the help of vxlan which in-turn uses various protocols to connect, which is beyond TCP/IP. Docker swarm uses UDP as per the ports required to open but I am also unknown about the overall architecture. I just wanted to know what overlay was.

### Setup
I set up the docker swarm environment using Vagrant which uses virtual box to initiate the vms.
All the vms need to be in the same cidr/subnet and Docker daemon must be setup in each of them.
Below is the Vagrantfile used:
```

Vagrant.configure("2") do |config|
  config.vm.define "manager" do |manager|
    manager.vm.box = "ubuntu/jammy64"
    manager.vm.hostname = "manager"
    manager.vm.network "private_network" , ip: "192.168.33.10" 
    manager.vm.provision "shell" , inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install docker docker-compose -y
    SHELL
  end

  config.vm.define "worker" do |worker|
    worker.vm.box = "ubuntu/jammy64"
    worker.vm.hostname = "worker1"
    worker.vm.network "private_network" , ip: "192.168.33.11"
    worker.vm.provision "shell" , inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install docker docker-compose -y
    SHELL
  end

  config.vm.define "worker2" do |worker2|
    worker2.vm.box = "ubuntu/jammy64"
    worker2.vm.hostname = "worker2"
    worker2.vm.network "private_network" , ip: "192.168.33.12"
    worker2.vm.provision "shell" , inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install docker docker-compose -y
    SHELL
  end

end
```
Using the vagrant up command will launch all the vms at the same time.

### Steps
---
- After the vms are up and running, ssh to all of them and in manager node execute the docker swarm mode using:
```
docker swarm init --advertise-addr 192.168.33.10
#here keep the ip address of the manager node
```
- The command will give a join token which should be copied and paste in the worker node.
<img width="570" alt="Screen Shot 2025-01-23 at 3 41 31 PM" src="https://github.com/user-attachments/assets/30a71889-3500-4029-bd9f-75b7339a7026" />

- Connecting the worker node:
<img width="570" alt="Screen Shot 2025-01-23 at 3 42 56 PM" src="https://github.com/user-attachments/assets/7074b39f-fce9-4232-b360-8109099158e6" />

- Now use the swarm command to check the status of the nodes.
  
  ```
  docker node ls
  ```
  
<img width="570" alt="Screen Shot 2025-01-23 at 3 44 18 PM" src="https://github.com/user-attachments/assets/b21462bb-0394-47c3-b210-29289f383b37" />

- Now all the nodes are active and running, the setup is successfull.
- In order to maintain the quorum of the swarm cluster, we can promote the worker nodes to be managers but on line.
- This is done because in case of failure in a manager, the in-line worker nodes can be a manager themselves.

- Now to test the working of the cluster, a simple service is created:

```
docker service create --name web_cluster --replicas 3 -p 80:80 nginx
```
- The service is up and running:
  
<img width="570" alt="Screen Shot 2025-01-23 at 3 51 28 PM" src="https://github.com/user-attachments/assets/3ae1fde9-6b97-4831-9a20-c8028e8d02d1" />

- Further to see either the tasks are equally distributed across all the nodes use:
```
docker service ps <service_name>
```

<img width="570" alt="Screen Shot 2025-01-23 at 3 53 18 PM" src="https://github.com/user-attachments/assets/82277363-dbf9-4d43-8140-52ad8e3861e9" />

- As we can see the tasks are equally distributed across all the nodes.

- Now we can experiment and use the swarm cluster as per our liking and to dive deeper we can always check the [docs](https://docs.docker.com/reference/cli/docker/service/create/).
