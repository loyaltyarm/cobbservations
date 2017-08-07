+++
date = "2017-07-29T16:36:59-04:00"
draft = false
title = "Bootstrapping macOS for CI"

+++

## Introduction
I've been working on some other posts about _provisioning_ macOS for the purposes of iOS CI. But before we can talk about provisioning, we might want to think about bootstrapping our macOS system first to make provisioning easier. What do I mean by bootstrapping, how is it accomplished, and why should you do it?

**Bootstrapping, in this case, refers to the initial setup of a macOS machine (could be non macOS also) in order to enable provisioning.** There are many reasons I've taken this approach in environments I support, but the main reason is that I prefer to use a _no-image_ workflow to set up my infrastructure. Additionally other admins are considering new workflows as well. At [MacDevOpsYVR](https://www.macdevops.ca) 2017, Michael Lynn talked about how Apple may be changing traditional system administration workflows in his talk, [Mac-narök: The End Times of Our Workflows](https://www.youtube.com/watch?v=4Qv8CE6D_Rc&list=PLOpBG-mD9ZjHdx6_yF4y8IR6eGDWgkJr9&index=1&t=2s). The talk is worth a watch for those considering retaining an imaging-based or more traditional setup workflow. But why do I do it this way?

## A no-image approach
No-image setups help me scale CI infrastructure quickly, and allow me to use macOS out of the box as it ships. This is important because I do not have to maintain imaging servers, large image repos, and other imaging infrastructure to support my growing fleet. I also don't have to spend time maintaining even a base thin image, which would periodically have to be updated at least when major macOS releases come down. Instead, I can put all of my bootstrapping logic into a script, and run that script on the machine after the Apple hardware is initially installed by a data center team. 

The other advantage this allows me is that I can move the business logic of my infrastructure setup amongst various pieces of hardware we might use, particularly Mac Pro or Mac Mini. I can even use the same bootstrap script for virtual machines too! I can also easily change provisioning tools if I need to, and install additional components by making a quick code change to my bootstrap script. Those changes are stored in version control and are reviewed before landing as well. Lastly, the time to set up new infrastructure is much faster, and less prone to network failures I've seen when trying to image large groups of macOS machines. Bootstrap+provisioning workflows allow me to move easily across data centers as well, because I only need reachability to my `git` and `munki` servers.

## Bootstrapping
What kind of things do we want in a bootstrap script? The list below gives some good examples:

- adding a specific administrator user account
- enable remote login (SSH) and VNC
- setting power management settings we need for CI (always on in our environment)
- installing configuration management tooling (Puppet, Chef, etc.)
- installing base dependencies (homebrew, munki tools, etc.)
- checking into a packaging server (for me, how I install Xcode)

After the list above, we would ideally use configuration management (Puppet, Chef, Ansible, etc.) to continue with any other provisioning, such as installing specific homebrew formula, pip packages, or ruby gems (dependencies).

## A Working Example
In my environment, I'm lucky enough to receive infrastructure configured by our data center team that includes the following:

- required administrator user account that has been logged in
- remote login (SSH) enabled

This makes it easy to ssh into a machine and run the `bootstrap.sh` script I use for my environment. To help, I've shared my `bootstrap.sh` [script on Github](https://github.com/loyaltyarm/scripts/blob/master/mac/bootstrap.sh). Let's take a look at each part below:

>Note, we have to run this script as our `root` user, because we change many system settings here in addition to installing user-space tools. You'd run this as `sudo ./bootstrap.sh` and the specification of the `USER` in the script would handle configuration of user space items automatically.

First, we `set -e` to exit immediately if any of the commands fail:

```
set -e
```

Then, we assign some variables for `brew` path, the [Munki client identifer](https://github.com/munki/munki/wiki/Preferences) (so that our machine receives only packages scoped to the bootstrap workflow), the Munki server URL, our desired Puppet version, our token administrator CI account, a quick and dirty way to identify the Wi-Fi interface (for disabling Wi-Fi), and our working directory for executing Puppet provisioning at the end of the script.

```
BREW="/usr/local/bin/brew"
CLIENT_IDENTIFIER="machine-bootstrap"
PACKAGING_SERVER_URL="http://munki-internal01.example.com/repo" 
PUPPET_VERSION="4.1.0"
USER="testuser"
WIFI_INTERFACE=`/usr/sbin/networksetup -listallhardwareports | awk '/Hardware Port: Wi-Fi/,/Ethernet/' | awk 'NR==2' | cut -d " " -f 2`
WORKING_DIRECTORY="/tmp/my-puppet-repo-folder"
```

Next, we download and install the Munki tools. This used to be a manual step because the munki tools [required a restart](https://github.com/munki/munki/issues/5), but because we will eventually restart the machine anyway, this is a non-issue.

```
echo "Installing the munki tools..."
pushd /tmp
/usr/bin/curl -OL https://github.com/munki/munki/releases/download/v3.0.3/munkitools-3.0.3.3352.pkg
sudo /usr/sbin/installer -pkg munkitools-3.0.3.3352.pkg -target /
popd
```

Then, we enable remote login (SSH):

```
echo "Enabling ssh..."
/usr/sbin/systemsetup -setremotelogin on
```

enable screen sharing (VNC):

```
echo "Enabling screen sharing..."
/System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -access -off -restart -agent -privs -all -allowAccessFor -allUsers
```

disable sleep and set _Always On_ power settings:

```
echo "Disabling sleep..."
/usr/sbin/systemsetup -setharddisksleep Never

echo "Setting power management settings..."
/usr/bin/pmset sleep 0
/usr/bin/pmset displaysleep 0
/usr/bin/pmset disksleep 0
/usr/bin/pmset womp 1 # wake on magic packet
/usr/bin/pmset powernap 0
/usr/bin/pmset autorestart 1 # autorestart on power failure
```

disable screen lock:

```
/usr/bin/defaults write com.apple.screensaver askForPasswordDelay 1
```

disable wireless (because we are wired in the data center):

```
echo "Disabling wireless..."
/usr/sbin/networksetup -setairportpower $WIFI_INTERFACE off
```

mute the sound (purely for sanity, nothing really special here):

```
echo "Muting the sound..."
/usr/bin/osascript -e 'set volume output muted true'
```

check for and install homebrew if missing:

```
if [ -f "$BREW" ]
then
    echo "Homebrew found."
else
    echo "Homebrew not found."
    sleep 1
    echo "You need to install homebrew!"
    sleep 1
    echo "We will do that now..."
    sleep 2
    sudo -u $USER /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" \ </dev/null
fi
```

for sanity we also update homebrew after installing:

```
sudo -u $USER /usr/local/bin/brew update
```

and install `brew cask` and `java` as dependencies for working with Jenkins:

```
sudo -u $USER /usr/local/bin/brew install caskroom/cask/brew-cask
sudo -u $USER /usr/local/bin/brew cask install java
```

then set some Munki properties:

```
echo "Setting munki properties..."
/usr/bin/defaults write /Library/Preferences/ManagedInstalls ClientIdentifier $CLIENT_IDENTIFIER
/usr/bin/defaults write /Library/Preferences/ManagedInstalls SoftwareRepoURL $PACKAGING_SERVER_URL
```

At this point in the script, we have set up all of the prerequisites needed to use Munki, so we'll go ahead and check in now with the _client identifier_ we used above:

```
echo "Checking for packages..."
/usr/local/munki/managedsoftwareupdate -v --checkonly
```
and then install any scoped packages:

```
echo "Installing packages..."
/usr/local/munki/managedsoftwareupdate -v --installonly
```

Now that the munki part of our workflow is finished, we'll install Puppet using the public gem (could also use a `.pkg` here). Note that because we are using the gem, we have to use the `-n` option to set the installed path to `/usr/local/bin`:

```
echo "Installing Puppet..."
sudo /usr/bin/gem install -n /usr/local/bin puppet -v $PUPPET_VERSION --no-rdoc --no-ri
```

We use Hiera-eyaml for secret management, so we'll install that as well:

```
echo "Installing Hiera-Eyaml..."
sudo /usr/bin/gem install -n /usr/local/bin hiera-eyaml --no-rdoc --no-ri
```

and install [r10k](https://docs.puppet.com/pe/latest/r10k.html) as well:

```
echo "Installing r10k..."
sudo /usr/bin/gem install -n /usr/local/bin r10k --no-document
```

This completes the Puppet installation, so we'll complete our final direct install of the last dependency, `fastlane`. [Fastlane](https://fastlane.tools/) is a great tool for generally managing various facets of iOS CI at scale; if you're not using it already you absolutely should be:

```
echo "Installing fastlane..."
sudo /usr/bin/gem install -n /usr/local/bin fastlane --no-rdoc --no-ri
```

So that's it--we've bootstrapped our infrastructure in one easy to use script! Did we forget anything? Actually yes...we haven't _run_ Puppet yet! There are many ways to do this, but the general masterless workflow looks like below:

- Clone your Puppet repo from source control
- Call the `puppet` binary directly using the `apply` command __OR__ write a wrapper script for `puppet` that performs both of these actions together

If you are using a puppet master, then you can add your puppet execution command into the end of the script to finish any reminaing configuration. In my environments, I prefer to run _masterless_ Puppet, a topology of which there are some benefits and tradeoffs. I recommend doing research if you're interested in using Puppet and exploring deployment types. 

There's a couple of other final steps that would finish our bootstrapping workflow. One of those could be getting the machine rebooted to help load any LaunchDaemons we've installed along the way and to make sure our infrastructure is fully ready to go. Another could be running the `softwareupdate` command to set updates to occur automatically or not at all (manually). Manual updates could be used since we have Munki here, and can deploy specific releases after testing them first against our tooling.

## Conclusion
Bootstrapping is an essential replacement of a no-image workflow, something that more shops should think about adopting, in my opinion. Being able to store your scripts in source control and migrate easily between tooling keeps your team flexible to changes by Apple or others, and gives you versioning to your configurations as well.
