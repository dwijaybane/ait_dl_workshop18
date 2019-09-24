# Setup on Paperspace

### steps to connect paperspace from local machine

0. Most important is as soon as you create machine on paperspace with public ip. You will get a password in your registered e-mail. Note that down. Also Note down your public ip somewhere.

1. Ensure Public Keys are available

- `cd` into `~/.ssh` directory
- if you don't have an `.ssh` directory in your home folder, create it (`mkdir ~/.ssh`)
- if you don't have an `id_rsa.pub` file in your `~/.ssh` folder, create it (`ssh-keygen` and hit Enter 3 times)

2. Add Paperspace info to `config` file
- if you don't have a config file, create one. This example creates file using gedit editor.
`$ gedit config`
- add these contents to your config file (replace IP address here with your Paperspace Public IP address)
```
Host paperspace
     HostName 184.###.###.###
     IdentityFile ~/.ssh/id_rsa
     # StrictHostKeyChecking no  
     User paperspace
     LocalForward 5000 localhost:5000
     LocalForward 8888 localhost:8888
```

3. `ssh` into Paperspace from local computer & change password
```
$ ssh paperspace   #provide mailed password for machine
$ passwd  #once done logout
```

4. Copy public key to Paperspace (Make login passwordless)

- Note: replace IP address in syntax below with your own public ip, and run command (provide your new password for machine)
`ssh-copy-id -i ~/.ssh/id_rsa.pub paperspace@184.###.###.###`
- Now you can simple login using `ssh paperspace`


### Connect to paperspace and setup our cloud machine
1. General setup
```
$ ssh paperspace
$ mkdir Downloads
$ cd Downloads
$ wget https://github.com/git-lfs/git-lfs/releases/download/v2.6.0/git-lfs-linux-amd64-v2.6.0.tar.gz
$ tar -xvzf git-lfs-linux-amd64-v2.6.0.tar.gz
$ sudo ./install.sh
$ rm *
$ git lfs install
$ git clone https://github.com/dwijaybane/dl-lab-files.git
$ du -sh dl-lab-files/  -> 41 M
$ cd dl-lab-files
$ ./nvidia-drivers.sh
```

Machine will shutdown. Go to your paperspace account and start your machine.
Now follow below steps.
```
$ ssh paperspace
$ cd Downloads/dl-lab-files
$ ./docker-setup.sh
```
Machine will shutdown again. So, go to your paperspace account and start your machine.

2. To test if docker works without root
```
$ ssh paperspace
$ docker run hello-world
```

Once we have docker running lets setup our data for examples
```
$  cd Downloads/dl-lab-files
$ ./extract.sh
$ ./download.sh
$ echo "LAB=~/Downloads/dl-lab-files/" >> ~/.bashrc
$ source ~/.bashrc 
```

3. Make sure docker_store exist (If not create if ref:docker-setup.sh)
`$ docker volume inspect docker_store`

4. Now we can download and run our dl-lab image in container
```
$ nvidia-docker run --rm --init -it --name container1 -p 5000:5000 -p 8888:8888 -v $LAB:/LAB -w="/LAB" -v docker_store:/docker_store streaminterrupt/dl-lab:base
  OR
$ nvidia-docker run --rm --init -it --name container1 -p 5000:5000 -p 8888:8888 -v $LAB:/LAB -w="/LAB" -v docker_store:/docker_store streaminterrupt/dl-lab:v3
```

5. Extract and prepare data set [In Container]
```
$ cd ObjectDetectionKITTI
$ ./prepare_kitti_data.py

#Delete data zip
$ rm -rf data_object_image_2.zip

$ cd .. #go back to LAB folder
```

6. To Run NVIDIA Digits [In Container]
`$ digits-devserver   #access on local browser localhost:5000`

7. Now open new tab of terminal
```
$ ssh paperspace
$ docker exec -it container1 bash
$ jupyter-notebook --ip=0.0.0.0 --port=8888 --allow-root  
```
access on local browser by ctrl + clicking link it generates

#### Note Everytime you make change in docker make sure to save image
Do it in new tab while your container is still active and required changes are done
`$ docker commit container1 streaminterrupt/dl-lab:base` 

### Useful Links:
- [For pushing docker images and saving it offline](https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html)

- [ssh port forwarding](https://stackoverflow.com/questions/37987839/how-can-i-run-tensorboard-on-a-remote-server)

- [Setup ssh for paperspace](https://github.com/reshamas/fastai_deeplearn_part1/blob/master/tools/paperspace.md)
