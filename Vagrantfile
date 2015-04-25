# Vagrant configuration to launch a Debian Jessie box with an seL4
# toolchain.

USER = ENV['SCHEDUL4_USER'] || `git config --global user.name`.strip
EMAIL = ENV['SCHEDUL4_EMAIL'] || `git config --global user.email`.strip

Vagrant.configure(2) do |config|
  config.vm.box = ENV['SCHEDUL4_VM_PROVISION'] ? 'deb/jessie-amd64' : 'benesch/schedul4'
  config.vm.hostname = 'schedul4'
  config.vm.provision 'shell', inline: <<-EOS.gsub(/^    /, '')
    set -o errexit
    outdated_seconds=$((60 * 60 * 24 * 2))
    uptodate() { (($(date +%s) - 10#$(stat -c %Y "$1" 2> /dev/null) < $outdated_seconds)); }

    export DEBIAN_FRONTEND=noninteractive
    dpkg --add-architecture armhf
    sed --in-place --regexp-extended \
      's|http://[^/]+/debian|http://httpredir.debian.org/debian|' \
      /etc/apt/sources.list
    uptodate /var/cache/apt/pkgcache.bin || apt-get update --assume-yes
    apt-get install --assume-yes \
        build-essential \
        cabal-install \
        ccache \
        gcc-arm-none-eabi \
        gcc-multilib \
        ghc \
        git \
        lib32gcc1 \
        libghc-missingh-dev \
        libghc-split-dev \
        libxml2-utils \
        ncurses-dev \
        python-jinja2 \
        python-pip \
        python-ply \
        python-tempita \
        qemu \
        realpath
    uptodate ~/.cabal/packages/hackage.haskell.org/00-index.cache || cabal update
    cabal install --global \
      MissingH \
      data-ordlist \
      split
    pip install --upgrade pip
    pip install \
      pyelftools \
      ply \
      jinja2
    if [[ ! -f /usr/local/bin/repo ]]
    then
      curl --silent https://raw.githubusercontent.com/android/tools_repo/v1.12.13/repo \
        > /usr/local/bin/repo
    fi
    sha256sum --check <<EOF
      07aa40b0508a8b7edf92f2073b4adcb6ec95f825accda72ad42f1e1b6d8e4881  /usr/local/bin/repo
    EOF
    chmod a+x /usr/local/bin/repo
    sudo -u vagrant git config --global user.name #{USER.shellescape}
    sudo -u vagrant git config --global user.email #{EMAIL.shellescape}
    mkdir -p /vagrant/project
    cd /vagrant/project
    repo init -u https://github.com/benesch/schedul4-manifest
    repo sync --jobs=25
  EOS
  config.ssh.shell = 'bash'
end
