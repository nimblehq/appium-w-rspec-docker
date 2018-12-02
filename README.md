# Docker image for Appium (and Node) + Rspec (Ruby) + ADB (Android SDK)
- Forget the pain of setting up node, appium, ruby, rvm...etc before being able to write test with [Appium](http://appium.io/), Docker is here to the rescue. Now you just need to install Docker components and start running your test suites without the hesitate of installation (which might be a bit unfamiliar to Mobile Developers, maybe not familiar with Docker too, but this setup is promised to be less error prone and easy to replicate!).

## This docker image includes:
  - Node v7.10.1
  - Appium v1.10.0
  - Ruby 2.5.1p57 (with rbenv)
  - Rspec (3.7.0)
  - ADB (with android build tools v28.0.0)
  
## Overview diagram
<p align="center">
  <img src="./snapshot-images/diagram.jpg" width="600"/>
</p>

## Usage:
### 1. Get all installation packages:
   - Docker Engine (https://docs.docker.com/docker-for-mac/install/)
   
   - If you are going to test with emulator from the host machine (I believe that most of us when start approaching this would follow this kind of setup), you might also need:
     - --Docker Machine (This is required for MacOS (and Windows also I suppose), important for routing connection from container to Android emulator).--
     - --VirtualBox (https://www.virtualbox.org/wiki/Downloads - for docker machine setup)--
     - Emulator (x86) image (or Genymotion is a good choice too).

### 2. Installation instructions:
   - Install Docker Engine
   - Clone and cd to this Repository.
   
   - Start building the Docker image: 
     
     `$ docker build . --tag appspec:latest` // change the image name and tag as you wish --tag [image_name]:[version]
     
   - When everything is done, Run a container from that image:
     
     `$ docker run --privileged -d -p 4723:4723 -e REMOTE_ADB=true -e ANDROID_DEVICES=10.0.2.2:5555 --name appspec-container -i appspec:latest` 
     // change the container [name:version] as you wish, simply replace appspec:latest
     
   - Arg explanation: 
      - Port 4723 is opened for Appium connection
      - ANDROID_DEVICES is the IP of the host emulator, with our current setup, the emulator is supposed to run on host machine at port 5555. `10.0.2.2` is [a special designed loopback address](https://developer.android.com/studio/run/emulator-networking) to help communication on the same host machine easier.
      - At init, the container will automatically provision for the emulator connection and starting Appium server.
   - Testing: 
      - Check if appium server is up: curl or access via browser: http://localhost:4723
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
