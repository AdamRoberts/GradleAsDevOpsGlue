## Deployments
* Developer Testing
* Acceptance Testing (UAT)
Note:  So with deployment automation finished we could move on to the next challenge, of how to actually do the deployments.
The two main types of deployments.  Developer deployments are most concerned about cycle time from modified code to running a test in the product with it.  Where Acceptance test deployments are more concerned about keeping the manual effort as low a possible, so if it takes several hours of computer time, but no human intervention it is a win.


## Developer Deployments
Note: UI developers have it easy, then can just run the gradle build on their laptop to setup and environment to test with..
But for backend developers we need to be able to develop and test against a mulithost clustered deployment as that is how the majority of our On Premise customers deploy the product.  And a lot of the interesting edge cases don't show up in a single host deployment.


#### Developer Deployments
## [![Oracle VirtualBox](images/VirtualBox.jpg)](https://www.virtualbox.org/)
Note:  Most of you are probably already familiar with VirtualBox, but for anyone that isn't, I software that allows running virtual machines on any x86 hardware.  I am not a lawyer, but the last I checked the license permits free usage for local use.  So don't try and host a server without paying but for develpment testing it is fine.  Of course if you are running VMs from you laptop you need a beefy machine.


#### Developer Deployments
## [![Hashicorp Vagrant](images/Vagrant.png)](https://www.vagrantup.com/)
Note:  Vagrant is developer productivity tool sepecifically designed to lower setup time for development environments.  I can automate creating a new VM in VirtualBox or other hypervisor and then running custom provisioning scripts.


#### Developer Deployments
## Vagrant Example
    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      config.vm.box = "centos-6.5-base"
      config.vm.box_url = "http://example.org/path/centos-6.5-base.box"
      config.vm.synced_folder "../mountpoint", "/vagrant"
      config.vm.provider :virtualbox do |vb|
        vb.customize [ "modifyvm", :id, "--memory", "2048" ]
      end
      config.vm.define "node1", primary: true do |worker|
        worker.vm.provision :shell, inline: '/vagrant/scripts/firstNode.sh'
      end
      config.vm.define "node2" do |worker|
        worker.vm.provision :shell, inline: '/vagrant/scripts/workerNodes.sh'
      end
    end
Note:  Vagrant needs a base "Box" to start with with is just a VM template with certain know user permissions.
This is a simple example Vagrant script that will start two nodes on the local VirtualBox.
Each node is provisioned by a script, the first node's script just calls the Gradle build with the deployment automation in it.
The Gradle build is not copied to the VM it is checked out to a shared directory that is mounted by Vagrant to both nodes.
This glosses over the problems of DNS configuration as that is dependent on your companies environment.


#### Developer Deployments
## [![Hashicorp Packer](images/Packer.png)](https://www.packer.io/)
Note:  Vagrant provides a starter box you can try that is an empty install of Ubuntu.
But we don't use Ubunt, and we wanted to load a basic set of applications that our product needs, such as a JVM.
Packer is product by the same company as Vagrant that takes your script and an ISO image of the installer and
sets up a new VM from scratch, that you can then create a new Vagrant Box from.
For use the custom box saves about 30 minutes of setup time for a new environement.


#### Developer Deployments
## ![Development Environment](images/DevelopmentEnv.png)
Note:  Here is the way all of the components we just discussed connect together.
We have the CentOs iso in the product repository, when we run Packer it pulls the iso, creates the nwe Vagrant Box and uploads it into the Product Repository.  The developer clones the repositories that contain the Vagrant scripts and the Deployment Automation to their machine and when Vagrant is run it pulls the base Box produced by Packer from the repository and sets up the VM.  Then the script provisions the VM which runs the Gradle build.  The Gradle build pulls the Product install and the Data files from the Repository and sets up the Product on the VMs.  Now the developer can make code changes, compile them then quickly copy those changes to the local VMs for testing.


## UAT Deployments
Note:  With all of the work we have already discussed there isn't much different for the UAT deployments other than coordination.


#### UAT Deployments
## [![Jenkins](images/Jenkins.png)](https://www.jenkins.io/)
Note:  We are currently using Jenkins to trigger our UAT deployments, nothing very special there.


#### UAT Deployments
## Agent = Customer Env
Note:  Where it differs is how we configured the Agents.  Commonly Jenkins agents are very similar and which agent is used doesn't matter that much.  But for us each Agent is a specific combination of Operating System, Database and OLAP Engine that matches a customer deployment.


#### UAT Deployments
## Acceptance Testing
Note:  deploying for UAT without running any tests isn't very useful.


#### UAT Deployments
#### Acceptance Testing
## [![CucumberJVM BDDs](images/Cucumber.png)](https://github.com/cucumber/cucumber-jvm)
Note:  So we added a simple Gradle tasks that can run the CucumberJVM BDD tool.  In this case the task does nothing but extend the built in JavaExec task with hardcoded values for most of the configuration.


#### UAT Deployments
#### Acceptance Testing
## ![UAT Task Graph](images/UAT-TaskGraph.png)
Note:  So now we just add the BDD test tasks as dependent on the Deployment tasks to give Jenkins a single task to call to run the whole testing suite.


#### UAT Deployments
## ![UAT Environment](images/UATEnv.png)
Note:  So in this case it is pretty simple, Jenkins triggers a build on an agent, by calling Gradle on that agent the Product is deployed, configured, dataloaded, and tested.  The automation comes from VCS, the product and data come from the repository.

