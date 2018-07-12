# Docker image for Appium (and Node) + Rspec (Ruby) + ADB (Android SDK)
- Forget the pain of setting up node, appium, ruby, rvm...etc before being able to write test with [Appium](http://appium.io/), Docker is here to the rescue. Now you just need to install Docker components and start running your test suites without the hesitate of installation (which might be a bit unfamiliar to Mobile Developers, maybe not familiar with Docker too, but this setup is promised to be less error prone and easy to replicate!).

## This docker image includes:
  - Node v7.10.1
  - Appium v1.8.1
  - Ruby 2.5.1p57 (with rbenv)
  - Rspec (3.7.0)
  - ADB (with android build tools v28.0.0)
  
## TODO: Overview & architecture

## Usage:
### 1. Get all installation packages:
   - VirtualBox (https://www.virtualbox.org/wiki/Downloads)
   - Docker Engine (https://docs.docker.com/docker-for-mac/install/)
   - Docker Machine (important for routing connection from container to Android emulator, already included in Docker installation bundles for Mac version).
   - Emulator x86 img or Genymotion is a good choice too.

### 2. Installation instructions:
   - Install the VirtualBox.
   - Install Docker Engine & Docker Machine from the package
   - Create a docker machine with default opts:
     
     `$ docker-machine create --driver virtualbox default `
     
   - Check for the IP address of that machine (note that the assigning IP step could take some time).
   - The Docker machine IP could be: **192.168.99.100** and the host machine is assigned to **192.168.99.1**
   - Clone and cd to this Repository.
   - Make sure the next following steps are executed on the Docker Machine domain: 
   
      `$ eval ($docker-machine env default)` // default is the docker machine name.
      
      *tips: you will need to run this every time opening a new Terminal tab, so load it to your bash profile to get rid of repeating.
   - Start building the Docker image: 
     
     `$ docker build . --tag appspec:latest` // change the image name and tag as you wish --tag [image_name]:[version]
     
   - When everything is done, Run a container from that image:
     
     `$ docker run --privileged -d -p 4723:4723 -e REMOTE_ADB=true -e ANDROID_DEVICES=192.168.99.1:5555 --name appspec-container -i appspec:latest` 
     // change the container [name:version] as you wish, simply replace appspec:latest
     
   - Arg explanation: 
      - Port 4723 is opened for Appium connection
      - ANDROID_DEVICES is the IP of the host emulator, with our current setup, the emulator is supposed to run on host machine at port 5555.
      - At init, the container will automatically provision for the emulator connection and starting Appium server.
   - Testing: 
      - Check if appium server is up: curl or access via browser: http://192.168.99.100:4723
      - Check if the adb inside container has connected to the emulator: 
        
        `$ docker exec -it appspec-container adb devices` 
        -> there should be a list showing that specific emulator connected (via its IP address).

### 3. Run your Rspec test:
   - The current setup might have missed some Android SDK tools, but intentionally we won't provide much Android dependencies here, so **make sure you had built APK** for testing first.
   - Copy the Project into this container: 
     
     `$ docker cp path/to/your/repo appspec:/project-name`

   - From here, you can either get INTO that container to execute command, or appending 
     
     `$ docker exec -it appspec [your_command]`
   
   - Get into the container with bash shell open: 
   
     `$ docker exec -it appspec /bin/bash`
   
   - Now you're inside the container, navigate to your project that contains the Gemfile.
   
     `cd path/to/your/workspace/repo`
   
   - Install your required gem: 
     
     `$ bundle install`
   
   - Run your specs: 
     
     `$ bundle exec rspec spec/your-specs`
   
   - NOTE: if your test suites has command to run with `adb` somewhere (e.g: adb install, adb uninstall...), make sure you have specify with `-s $ANDROID_DEVICES` to route the command to the specific emulator you want.


### Thoughts?
