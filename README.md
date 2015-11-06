[![Deploy to Bluemix](https://bluemix.net/deploy/button_x2.png)](https://bluemix.net/deploy?repository=https://github.com/joshisa/piwikstart)
###Piwik::Self-Assembly
<i>noun</i>
 1. The spontaneous formation of a body in a medium containing the appropriate components
 2. The rapid instantiation of a [Piwik](http://piwik.org/ "Piwik") Deployment Instance on IBM Bluemix containing the appropriate components.

An opinionated one-click self-assembling deployment of the Piwik platform for web and traffic analytics onto a CloudFoundry platform. A number of marketplace plugins are automatically installed and available for activation. Twilio and Sendgrid CloudFoundry based service support is baked in.  

#### Why?
Open source projects are awesome. PaaS CloudFoundry enabling of self-hosted open source application platforms is messy.  Making a mashup between cool opensource and cloud-enabling tweaks that makes deployment feel sweet and simple is "hard to do".  Legal review burdens aside, the level of ongoing maintenance effort is directly proportional to the number of tweaks in the mashup.  So, keeping a repository concise and abbreviated in content is smart.  My objective with this repo experiment is to facilitate consistent, rapid Piwik deploys on IBM Bluemix with minimal deployment friction using the fewest files possible.

#### Getting Started  (Pre-requisite: [CF CLI](https://github.com/cloudfoundry/cli/releases "CF CLI"))
- Click the Deploy Button Above.  Verification Point:  Await success of all 4 steps on the deploy page.
 
![Step 1](https://github.com/joshisa/piwikstart/blob/master/bluezone/img/step1.png)

- Click on the 2nd step (Cloned Repository Successfully).  Click on the Edit Code button in the upper right to establish a git repository for your newly created project.  There is a **git** section that should contain your repo url.  
![Git URL Section](https://github.com/joshisa/piwikstart/blob/master/bluezone/img/giturl.png)
- Clone the repo of the IBM DevOps repository to your local machine.  
  ``` git clone https://hub.jazz.net/git/<your_id>/<app_name>/ ``` 
- Browse to the web installation url @ https://replace_me_with_app_name.mybluemix.net/ and complete the multi-step process.  During the system check phase, it is normal to see a file integrity check warning message.  This is caused by:
  - movement of composer config files
  - missing .gitignore files
  - detection of web installer tweaks to reduce end user friction
- {OPTIONAL} At this point, you may choose to login and browse to the administration section of Piwik.  
  **NOTE**: To encourage better security practices, the deploy is configured to only allow login via **HTTPS**.  Attempting to login via non-SSL will result in a **Form Security failed error**.  Within the plugins section, you will be able to **activate** or **deactivate** plugins of your choice.  Your choices will be persisted within a generated file named **config.ini.php** that we will need to pull down and persist back into the repository.  As a cloud-enabled 12 factor application, the app's local file storage is ephemeral.  Without persistence, any restart or crash/restart sequence will cause your Piwik application to revert back to the web installer sequence.
- Within the terminal, browse to the root dir of your local cloned repo and execute a command similar to:
```
$ cf files <replace_me_with_app_name> /app/fetchConfig.sh | sed -e '1,3d' > fetchConfig.sh
$ chmod +x fetchConfig.sh
```
- This should pull down a helper bash script similar to what's shown below.  Your app's name will already be populated :-)
```
#!/bin/bash
rm ./bluezone/configtweaks/config.ini.php
cf files <replace_me_with_app_name> /app/htdocs/piwik/config/config.ini.php > ./bluezone/configtweaks/config.ini.php
sed -i -e '1,3d' ./bluezone/configtweaks/config.ini.php
```
- Perform a git add, git commit and git push to persist the config.ini.php within the IBM DevOps repository. For example,
```git add -A```
```git commit -m "Persisting the installer wizard generated config.ini.php"```
```git push```
- With this git commit, your IBM DevOps pipeline will retrigger and re-deploy Piwik.  In a few short minutes, your Piwik application will be ready for you to use.

#### How is this better than Docker?
Better is not the right question.  Sometimes a cool project may not have a high-quality Dockerfile image available yet.  Sometimes you may want to learn more about how specific runtimes and buildpacks behave within a PaaS - so what better way than to see in detail how complex apps and platforms are assembled for PaaS CloudFoundry cloud deployment.  Sometimes you feel more comfortable understanding <<fill in your favorite runtime language - PHP, Python, Node.js, ...>> than Docker CLI.  Sometimes Docker is the way better approach to go, but you like doing things the hard way ;-)

#### How does it work?
The magic is in the pipeline.yml.  A build script is embedded which precisely defines the steps required to PaaS CloudFoundry enable the opensource application.  The script pulls code from various locations, applies tweaks and cleans itself up into a deployable asset.  Using IBM DevOps services, you may download the built asset (which can then be tweaked further and deployed manually using the CF CLI) or simply let the DevOps pipeline continue to do the assembly and deploy effort for you.  The former is useful for devs looking to innvoate and expand capabilities of the open source project (For example: adding cognitive computing interactions from something like IBM Watson) and the latter is for folks simply desiring a rapid turnkey deployment of the opensource project for end-use.

#### Why didn't you use packaging technologies like Composer, Bundler, ...?
In many cases, the deploy in fact is using these packaging technologies to help gather app platform dependencies. Complementary to that behavior, this repository automates the organization and customization of files PRIOR to dependency inspection and installation.  For example, customization tweaks that make the web installer process smarter in self-populating bound service credentials, new feature tweaks that include service client sdks, etc ...  

#### Installing additional plugins
If you want to add additional Piwik plugins not included within this repo,  download your plugin of interest from the [Piwik Plugins Marketplace](https://plugins.piwik.org/ "Piwik Plugins Marketplace").  Extract the zip contents into the folder **/bluezone/configtweaks/plugins** . This would result in something like ./bluezone/configtweaks/plugins/myawesomeplugin .  You are free to add as many plugin folders as you'd like within this parent **plugins** dir.  The scripts will loop through and place them within the correct location for you.
- Perform a git add, git commit and git push to persist the newly added plugins within the IBM DevOps repository. For example,
```git add -A```
```git commit -m "Added myawesomeplugin to my Piwik deploy"```
```git push```
- With this git commit, your IBM DevOps pipeline will retrigger and re-deploy Piwik.  You will then need to access the administration section of Piwik and **Activate** the myawesomeplugin.  Finally, you will need to re-fetch the config.ini.php file using techniques outlined above and re-persist this updated file within the repository.  As you can see, it is wise to plan ahead on plugins that you'd like to use in your deploy given the overhead involved in persisting their install and activation.

#### Reference
[Getting Started with Piwik on Bluemix](https://developer.ibm.com/bluemix/2014/07/03/getting-started-piwik-ibm-bluemix/)
