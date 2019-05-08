# Getting X11 GUI applications to work on OS X with Docker

This part of the project makes the XQuartz available for auto open by other applications (e.g. docker)

This should only have to be run if you see a message kinda like this:

```txt
(google-chrome:1): Gtk-WARNING **:  cannot open display: :0
```

e.g.

```text
(google-chrome:1): Gtk-WARNING **: 00:00:00.000: cannot open display: 192.168.42.42:0
```

Based on [this](https://gist.github.com/stonehippo/2c2b0972b7d199c78fb94fa9b1be1f5d) description.
I just went a step further to create a service out of it so that socat will be started auto-magically :-)

Sometimes it will fail though if the IP changes or something like that. Then you will see the message again and
you can solve it by just running this script again.

# Requirements

* [Homebrew](https://brew.sh) should be installed on your mac
* socat needs to be installed (`brew install socat`)
* XQuartz.app needs to be installed (`brew cask install xquartz`)

# Usage

## (re) install
must be run as root

```bash
sudo sh ./install.sh
```

## Unistall

```bash
sudo sh ./uninstall.sh
```

# Docker run script

Now if you want to add X11 functionality to your docker image.
It of course needs to be an X11 docker image :-)

but you should also add the following to your docker image startup script:

(Just an example Tested on macOs) 
```bash
# Adding the ip to the xhost
ip=$(ifconfig en0 | grep inet | awk '$1=="inet" {print $2}')
xhost + $ip

# If audio is also wanted for your X11 app (pulseaudio)
[[ -z "$(brew ls --versions pulseaudio)" ]] && brew install pulseaudio
killall pulseaudio 2>/dev/null
pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon 2>/dev/null

docker run                                            \
    -d                                                \
    --rm                                              \
    -v /tmp/.X11-unix:/tmp/.X11-unix                  \
    -e DISPLAY=$ip:0                                  \
    -e PULSE_SERVER=docker.for.mac.localhost          \ 
    -v ~/.config/pulse:/nobody/.config/pulse          \
    ivonet/x11-chrome
    if [ "$?" != 0 ]; then
        echo "you might want to see if the XQaurtz is still correctly configured. See http://ivo2u.nl/oB."
    fi

```

These two lines enable the X11 stuff. Note that the $ip is gathered above in the script

```bash
    -v /tmp/.X11-unix:/tmp/.X11-unix                  \
    -e DISPLAY=$ip:0                                  \
```

These two lines enable the audio

```bash
    -e PULSE_SERVER=docker.for.mac.localhost          \ 
    -v ~/.config/pulse:/nobody/.config/pulse          \
```
