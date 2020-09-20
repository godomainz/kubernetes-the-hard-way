{
    sudo apt-get remove docker docker-engine docker.io containerd runc 
    sudo apt-get update 
    sudo apt-get -y install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo apt-key fingerprint 0EBFCD88
    sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io
}

{
    sudo usermod -aG docker ${USER}
    su - ${USER}
    id -nG
    echo $(docker --version)
}

{
    sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose && echo $(docker-compose --version)
}


cat <<EOF | sudo tee /etc/docker/daemon.json
{
    "dns": ["8.8.8.8", "8.8.4.4"]
}
EOF
sudo service docker restart


{
    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
    sudo apt install -y ./google-chrome-stable_current_amd64.deb
}

{
    wget https://www.syntevo.com/downloads/smartgit/smartgit-20_1_4.deb
    sudo apt install -y ./smartgit-20_1_4.deb
}
