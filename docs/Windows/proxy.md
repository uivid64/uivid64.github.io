## How to use the web proxy on the terminal

### Error

```
'Connection to api.telegram.org timed out. (connect timeout=5.0)'
```

### Environment

macOS 11.6, using ClashX 1.71.0.4, ClashX normal proxy, using self contained terminal.

### Configuration

Open the terminal and enter the following code to enable the proxy.

```
cat > ~/.bash_profile << EOF
function proxy_on() {
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=\$http_proxy
    echo -e "Terminal proxy is enabled."
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "Terminal proxy is off."
}
EOF

source ~/.bash_profile

proxy_on
```

Turn off the terminal proxy by typing ``proxy_off`` in the terminal.

### Result

```
curl -I http://www.google.com
HTTP/1.1 200 OK
curl -I https://api.telegram.org
HTTP/1.1 200 Connection established
```

## Preface

When we use `Homebrew`, `git`, `npm` and other commands in the terminal, the installation always fails because of network problems.

Especially when installing `Homebrew`, I understand that many of you have spent a long time trying to solve it, and I don't know how many times you've complained about the damn network.

Although it is true that setting up a mirror is useful, it is not universal, so today we will introduce a way to let the terminal also take the proxy, which can kill many cases.

The proxy mentioned in the text refers to the case with `Clash`, other software also has some common ground.

## macOS & Linux

By setting `http_proxy`, `https_proxy`, you can make the terminal go through the specified proxy.
The configuration script is as follows and will only take effect temporarily when executed directly in the terminal.

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=$http_proxy
```

``7890`` is the port corresponding to ``http`` proxy, please don't copy the job, modify it according to your actual situation.

**You can find the proxy port information in Clash's settings screen. **

### Handy script

Here is a handy script that contains functions to turn on and off.

```
function proxy_on() {
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=\$http_proxy
    echo -e "Terminal proxy is turned on."
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "Terminal proxy is off."
}
```

Start the proxy with `proxy_on` and turn it off with `proxy_off`.

Next you need to write the script to `.bash_profile` or `.zprofile` so that it will be permanent.

> You may ask, how to write the script, be patient, the method to install the script is provided below.

As for which file you should write, please judge by the result of the command `echo $SHELL` which returns.

- `/bin/bash` => `.bash_profile`
- `/bin/zsh` => `.zprofile`

Then execute the **installation script** (append + take effect), note that the name of `.bash_profile` must be changed according to the above result: ``/bin/bash` => `/bin/zsh` => `.zprofile`

```
cat > ~/.bash_profile << EOF
function proxy_on() {
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=\$http_proxy
    echo -e "Terminal proxy is enabled."
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "Terminal proxy is off."
}
EOF

source ~/.bash_profile
```

Open the proxy

```
proxy_on
```

Turn off the proxy

```
proxy_off
```

You can execute ``curl cip.cc`` to verify: (clash on global)

```
IP : xxx
Address : Taipei City, Taiwan, China
Carrier : cht.com.tw

Data 2 : Taiwan Province | Chunghwa Telecom (HiNet) Data Center

Data 3 : Taiwan, China | Chunghwa Telecom

URL : http://www.cip.cc/xxx
````

I've seen online that `curl -I http://www.google.com` may encounter `403` problem, you need to pay attention to this situation when using `Google` domain verification.

## Windows

According to the article on the web, using the global proxy method under `Windows` will also work for `cmd` (without validation).

### cmd

```
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890
```

Restore command.

```
set http_proxy=
set https_proxy=
```

### Git Bash

Set up the same as "macOS & Linux"

### PowerShell

```
$env:http_proxy="http://127.0.0.1:7890"
$env:https_proxy="http://127.0.0.1:7890"
```

Restore command (not verified).

```
$env:http_proxy=""
$env:https_proxy=""
```

## Other proxy settings

### git proxy

```
## Settings
git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'

# Restore
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### Github Proxy

GitHub documentation , Releases , archive , gist , raw.githubusercontent.com file proxy accelerated download service.

#### **git clone**

**git clone** https://ghproxy.com/https://github.com/stilleshan/ServerStatus

#### **git clone private repository**

The Clone private repository requires users to request a Token from [Personal access tokens](https://github.com/settings/tokens) to use it.
**git clone** https://**user:your_token**@ghproxy.com/https://github.com/your_name/your_private_repo

#### **wget & curl**

**wget** https://ghproxy.com/https://github.com/stilleshan/ServerStatus/archive/master.zip
**wget** https://ghproxy.com/https://raw.githubusercontent.com/stilleshan/ServerStatus/master/Dockerfile
**curl -O** https://ghproxy.com/https://github.com/stilleshan/ServerStatus/archive/master.zip
**curl -O** https://ghproxy.com/https://raw.githubusercontent.com/stilleshan/ServerStatus/master/Dockerfile

### npm

```
### npm set

npm config set proxy http://server:port
npm config set https-proxy http://server:port

### npm config set proxy
npm config delete proxy
npm config delete https-proxy
```



### git clone ssh how to go proxy

#### macOS

Open `~/.ssh/config`, if you don't have this file, create it yourself manually.

```
# global
# ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
# Set for specific domains only
Host github.com
    ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
```

#### Windows

Open the `C:\Users\UserName\.ssh\config` file and create it manually if you don't see it.

```
# Global
# ProxyCommand connect -S 127.0.0.1:1080 %h %p
# Set for specific domains only
Host github.com
    ProxyCommand connect -S 127.0.0.1:6600 %h %p
```

## Do I have to set a separate proxy for git if I use a proxy on the terminal?

There are two protocols for git, one is https and the other is ssh.

If you are using https, you can set up a proxy for the terminal. If you are using ssh, you need to configure a separate proxy.