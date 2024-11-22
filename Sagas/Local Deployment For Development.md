# 2024-08-09
## 17:52 - Figuring out if I need plugins
- First thing to check is if most of the heavy lifting can be done by plugins.
- It's looking like phpmyadmin is the thing to use actually.

# 2024-08-10
## 08:21 - Researching phpmyadmin
- It's a web UI, it's not necessary for creating what I need actually.

## 08:36 - Duplicator
- Okay, I think this phpmyadmin ended up being a largely pointless thing to look at, instead I may as well try the plugin solution and see if it works out of the box. It's called [Duplicator](https://wordpress.org/plugins/duplicator/).
- It's got 40 million downloads apparently and everyone bums it, so it's probably pretty good. I'll give it a go.
- As I'm installing it I am getting many many more plugins now. At some point I'll probably have to get serious about which ones I need and whether I should be using all of them at once.
- Okay, so I downloaded the duplicator plugin and activated it.
- Then I've gone through the process of setting up, scanning and building my zip file.
- There was one issue where it was complaining that the size of the database was too big for `mysqldump` or something, but when it actually came to do the build it built fine and there were no issues. I don't know what that's all about, but whatever.
- Now, this has created an archive file on the site itself saved inside of wp-content, I'll probably delete this after the fact I think. But it's good to know where it all gets saved etc. and that it's accessible to someone with system administrator access.

## 08:54 - Getting the zip file into local
- So I've downloaded the Duplicator file type and it's a `.daf` file type, presumably to try and keep you locked in to their stupid system. So ridiculous, they want you to pay for pro to get the data imported into a new site.
- Fuck ttthhaattt!
- Instead, I'm going to try the wp staging plugin and see if that works.
- Also, I'm deleting the duplicator file from wp-content.
- Also also, I'm going to see if chatgpt knows anything about the .daf file type and if there's any easy way around it.
- Nope, ChatGPT doesn't know how to do it, so fuck it, I can't be bothered with that.
- I'm going to delete the Duplicator plugin and try wp staging.
- I deleted it and then thought, oh shit maybe that annoying `.daf` file is hanging around somewhere on the site now. So I'm going to check and see if that actually is the case or not.
- I used the following command via an SSH into the terminal:
```sh
find ./ -name *.daf
```
- It returned me nothing, therefore I think it's likely the duplicator deleted its data when I deleted it.

## 09:21 - WP Staging
- Weird, when I was running local but I'd altered my hosts file to change where app.mumie.health should point, local was still intercepting traffic and trying to send me to the localhost and failing! Mad.
- I'd basically commented out the re-directs in the hosts file automatically set up by local, but it still intercepted some how!
- So it's clearly listening out for the app.mumie.health in other ways as well as using the hosts file!
- I resolved the issue by killing local and then everything went back to being fine.
- Anyway I've installed the [wp staging](https://wordpress.org/plugins/wp-staging/) plugin.
- I've created a backup of the site and downloaded it. Time to see if it can be imported and used to restore my site.
- Okay, I've loaded locals again and its altered the hosts file, but again it needs the flushdns to work properly!
- It's strange that when you've already accessed the site in a browser it keeps the current DNS location, clearly, I guess it caches it within the browser session or something?
- Yeah the browser keeps a track of the DNS IP address locations during the session because it makes subsequent lookups a lot faster. This is really handy! And also explains the behavior when you change something in your local hosts file and it doesn't immediately update.
- Yet more bullshit from the WordPress plugin ecosystem. Someone has always got their hand out for doing a very basic thing for you in WordPress!
![[Pasted image 20240810094732.png]]
- Okay, I've deleted the backup now. So that's that done, I'll try another plugin now and see if that works.
- I've deleted the wp-staging plugin too.

## 09:55 - Backup Migration
- I'm now trying this plugin: https://wordpress.org/plugins/backup-backup/
- Apparently this version is free, so it's probably going to be a bit better. Although the size of the site that can be zipped is 2GB max.
- It's backing up now, the interface is utter dogshit, I must say.
- Okay, I backed up the site and found where the backup is stored, it's the smallest of all the backup sizes. LOL presumably because it's using standard zip and not some proprietary shitty algorithm to stop people doing stuff themselves.
- Anyway, I've got the file downloaded as a zip now, time to setup the local hosting again!
- If you change the host file ahead of time before running your site on local then local won't prompt you to give administrator access for it to change the site. Really helpful when you have to be an administrator to do this.
- Okay, I'm on the super fake certificate of app.mumie.health hosted locally.
- Let's see if this will finally work!
- Okay, I've got it ready to go! Am I going to be blue ballsed at the last minute this time...
![[Pasted image 20240810100947.png]]
- Okay, and it got through the extraction part, and then started being in this other part where it was hanging a fair bit:
![[Pasted image 20240810101135.png]]
- Finally, one that actually worked!
![[Pasted image 20240810102008.png]]
- I then got something telling me to update my WordPress Database, which I did, it took about 1 second and then that was it, now everything works apparently.

## 10:21 - Preparing the local site
- I immediately killed monster analytics so nothing was being sent to Google analytics that's bogus that might upset the true analytics.
- Other than that, I think everything works really well and I'm not worried about it hurray!
- Actually no, I've deactivated Yoast as well just in case that's doing something.
- I guess the emailing system isn't going to work either actually, I could test and see but I don't think it will.
- But yeah, I think that's it, this is something that could be written into a medium article, Jesus Christ. What an annoying waste of time!

## 19:33 - Continuing
- I should look at the other plugin that's available for this updraft #idea or some such bollocks. Probably worth checking out for an article I write about it to be complete.