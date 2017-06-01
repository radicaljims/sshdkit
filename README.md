# sshd-and-stuff-kit

## Background

Forked from: https://github.com/alexellis/sshdkit

Interesting blog posts:

1. http://blog.alexellis.io/boot-linuxkit-in-10-mins/
2. http://blog.alexellis.io/minio-linuxkit/

## Installing Linuxkit

1. Install go: `brew install go` will do on OSX
2. Install Linuxkit:
'''
https://github.com/linuxkit/linuxkit.git
cd linuxkit
make
cp ./bin/* /usr/local/bin
'''

## Mac Usage

Requirements: a recent OSX, a recent Docker Edge installation, Go (see 1. above)

1. Generate a bootable image based off of setup.yml
'''
./generate_image.sh
'''
2. Run it in Hyperkit/VPNkit
'''
./boot.sh
'''
3. TODO client stuff

## runc / containerd

'''runc list # show running containers'''
