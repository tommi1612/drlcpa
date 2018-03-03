# MiniWoB-starter-agent

This codebase implements an extension of the [universe starter agent](http://github.com/openai/universe-starter-agent) provided by OpenAI that is able to solve a number of [MiniWoB](http://alpha.openai.com/miniwob/) environments.

![MiniWoB](https://github.com/tommi1612/miniwob-starter-agent/raw/master/imgs/MiniWoB.jpg "MiniWoB")

### Dependencies

* [Python 3.5](https://www.python.org/downloads/release/python-350/)
* [Golang](https://golang.org/doc/install)
* [six](https://pypi.python.org/pypi/six)
* [TensorFlow](https://www.tensorflow.org/) 0.12
* [tmux](https://tmux.github.io/) (the start script opens up a tmux session with multiple windows)
* [htop](https://hisham.hm/htop/) (shown in one of the tmux windows)
* [gym](https://pypi.python.org/pypi/gym)
* [libjpeg-turbo](https://libjpeg-turbo.org)
* [universe](https://pypi.python.org/pypi/universe)
* [opencv-python](https://pypi.python.org/pypi/opencv-python)
* [numpy](https://pypi.python.org/pypi/numpy)
* [scipy](https://pypi.python.org/pypi/scipy)
* [Docker](https://www.docker.com)


### Getting Started

```
conda create --name universe-starter-agent python=3.5
source activate universe-starter-agent

sudo apt-get install -y numpy golang libjpeg-turbo8-dev make tmux htop cmake golang libjpeg-dev
sudo apt-get install -y python-numpy python-dev zlib1g-dev xvfb libav-tools xorg-dev python-opengl libboost-all-dev libsdl2-dev swig
pip install "gym[atari]"
pip install universe
pip install six
pip install tensorflow
conda install -y -c https://conda.binstar.org/menpo opencv3
conda install -y numpy
conda install -y scipy

sudo apt-get -y remove tmux
sudo apt-get install wget tar libevent-dev libncurses-dev
VERSION=2.6 && mkdir ~/tmux-src && wget -qO- https://github.com/tmux/tmux/releases/download/${VERSION}/tmux-${VERSION}.tar.gz | tar xvz -C ~/tmux-src && cd ~/tmux-src/tmux*
./configure && make -j"$(nproc)" && sudo make install
cd && rm -rf ~/tmux-src

pip install opencv-python

pip install numpy --upgrade --ignore-installed

```

Concluding it is important to build the [newDockerImage](https://github.com/tommi1612/drlrpa/tree/master/newDockerImage) and to replace the runtimes.yml file at the location where universe is installed with the [file](https://github.com/tommi1612/drlrpa/tree/master/universe/universe/runtimes.yml) inside this repository.

### MiniWoB ClickTest

`python train.py --num-workers 2 --env-id wob.mini.ClickTest-v0 --log-dir /tmp/clicktest`

The command above will train an agent on the [ClickTest task](http://alpha.openai.com/miniwob/preview/miniwob/click-test.html) through VNC protocol.
It will see two workers that will be learning in parallel (`--num-workers` flag) and will output intermediate results into given directory.

The code will launch the following processes:
* worker-0 - a process that runs policy gradient
* worker-1 - a process identical to process-1, that uses different random noise from the environment
* ps - the parameter server, which synchronizes the parameters among the different workers
* tb - a tensorboard process for convenient display of the statistics of learning

Once you start the training process, it will create a tmux session with a window for each of these processes. You can connect to them by typing `tmux a` in the console.
Once in the tmux session, you can see all your windows with `ctrl-b w`.
To switch to window number 0, type: `ctrl-b 0`. Look up tmux documentation for more commands.

To access TensorBoard to see various monitoring metrics of the agent, open [http://localhost:12345/](http://localhost:12345/) in a browser.

![tensorboardIdentifyShape](https://github.com/tommi1612/miniwob-starter-agent/raw/master/imgs/tbIdentifyShape.png "tensorboardIdentifyShape")

The VNC environments are hosted on the EC2 cloud and have an interface that's different from a conventional Atari Gym
environment;  luckily, with the help of several wrappers (which are used within `envs.py` file)
the experience should be similar to the agent as if it was played locally. The problem itself is more difficult
because the observations and actions are delayed due to the latency induced by the network.

More interestingly, you can also peek at what the agent is doing with a VNCViewer.

![ClickTestVNC](https://github.com/tommi1612/miniwob-starter-agent/raw/master/imgs/ClickTestVNC.png "ClickTestVNC")

You can use your system viewer as `open vnc://localhost:5900` (or `open vnc://${docker_ip}:5900`) or connect TurboVNC to that ip/port.
VNC password is `"openai"`.

Note that the default behavior of `train.py` is to start the remotes on a local machine. Take a look at https://github.com/openai/universe/blob/master/doc/remotes.rst for documentation on managing your remotes. Pass additional `-r` flag to point to pre-existing instances.

For best performance, it is recommended for the number of workers to not exceed available number of CPU cores.

You can stop the experiment with `tmux kill-session` command.

### Atari and Flashgames

The agent within this extension is still able to oeprate on atari environments. To enable the use of flashgames environments the flashgames.json file must be added to the location where universe is locally installed.


### Evaluation

To evaluate the performance of the algorithm it was compared to a random agent and a human playing solving the MiniWoB tasks. Since this method is not capable of NLP most of the environments had to be excluded. Furthermore some other environments turned out to be unstable. Therefore the evaluation of this tasks was impossible, too. After all there was 25 Environments remaining for evaluation. On these the agent was trained for 12 hours each unless it was finished beforehand.

![Evaluation](https://github.com/tommi1612/miniwob-starter-agent/raw/master/imgs/Evaluation_DRL_MiniWoB.png "Evaluation")

On 35% of the tested tasks the agent was able to compete with human performance, while it did even outperform a human player on 22% of the evaluated tasks. 

The logs of this experiments can be found [here](https://github.com/tommi1612/logs).