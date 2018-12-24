---
layout: post
title: Using the Smashing Dashboard
---

### Visibility

A useful tool in providing visibility of your systems, processes and business metrics is a dashboard. 

Smashing is a maintained ruby based open source dashboard, spawned off from the now retired dashing.

You can get the source from here
https://github.com/Smashing/smashing

### Windows and Ruby

Having tried to install ruby from here
https://rubyinstaller.org/downloads/

And followed the steps to create a smashing sample dashboard:

```gem install bundler
gem install smashing
smashing new 'project'
cd 'project'
bundle    
smashing start
```

Came across this problem:

```Unable to load the EventMachine C extension; To use the pure-ruby reactor, require 'em/pure_ruby```

One solution would be to build the eventmachine from source, another is to use docker. 

### Using Docker

Having already installed the docker toolbox and go that running a simpler and faster approach was to run up a docker image of smashing. This is one example, or you can build your own.

``https://hub.docker.com/r/visibilityspots/smashing/``

This uses the proven ruby build with smashing in alpine. No need to wrestle with getting a windows version to work.

You can then map the developement folders in windows to the docker volumes. 

In windows you'll have to share the folder first in virtualbox and use that name for the volume path. 

Like so:

```
docker run -d -p 3030:3030 -v /windowFolder/smashing_dashboard/dashboards:/dashboards --name smashing visibilityspots/smashing
```

Where /windowFolder would be mapped in virtualbox to the windows directory where the smashing source is.


You can then call it using 

http://localhost:3030

Which defaults to sample.








