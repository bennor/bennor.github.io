---
layout: post
title:  "Speed up Entity Framework Migrations with Splatting"
date:   2014-11-01 17:37:00
categories: entity-framework powershell
---
Entity Framework migrations are awesome&hellip; really awesome. They take most of the pain out of deploying updates to your database, and they're easy as hell to work with.

But if you're working in a big solution with a lot of projects, or if you have an MVC app with `MvcBuildViews` turned on, you're going to be sitting round for an eternity every time you run any migration command that touches the database.

While the down time might give you a great opportunity to [swordfight with your workmates][1], some of us have better things to do, and it can get pretty frustrating when you make a mistake and have to rescaffold a migration.

## So what is taking so long?

The problem lies in the fact that EF migrations operations build the startup project and the target project each time they run. 

This means that even after you've gone to the trouble of moving them out of your UI project and into their own assembly like a responsible adult, the entry point for each migrations command is still the startup project in your app, because that's where the config containing the connection string for your DbContext lives. The upshot is you're building a whole heap of stuff that has nothing to do with your data and wasting a whole lot of time in the process.

## And how can we make it faster?

Each migrations command gives you the option to specify a target project (where your context lives) and a startup project (where the config containing your connection string A lives). 

To get this to work, the first thing you're going to want to do is copy your connection string from your actual startup project, to an `app.config` file in the project where your `DbContext` lives.

Now you can tell EF to use this project as both your startup and target projects, by explicitly specifying the target and startup projects as part of the command:

```powershell
Add-Migration AddReasonsForCrowAwesomeness -TargetProjectName IHeartCorvids.Data -StartupProjectName IHeartCorvids.Data
```

So that's made things way faster. _Awesome._

Problem is, we're doing all this to make our lives easier, and typing all that out every time is going to get old about as quickly as waiting it to do its thing the slow way.

## Splatting to the rescue

Since we're doing all our migrations work in the Package Manager Console (or PMC for short), we have the full power of Powershell at our disposal. A lesser known feature of Powershell is the concept of "splatting". The way it works is very simple:

1. You create a map (about the same as an anonymous type), and assign it to a variable:
	
```powershell
$IHeartCorvids = @{TargetProjectName = "IHeartCorvids.Data"; StartupProjectName = "IHeartCorvids.Data"}	
```

2.  Now, you can "splat" the map variable into any of your migrations commands, like this:

```powershell
Update-Database -TargetMigration AddReasonsForCrowAwesomeness @IHeartCorvids
```

That's really all there is to it. The contents of the map that we stored in a variable in step 1, has all of its keys and values used as parameter names and values in the command we're running in step 2.

Note: Pay close attention to the use of the `@` in the splatting. A rookie mistake is to use the variable as-is (e.g. $myMapVariable), which won't splat it and will **???** instead.

## Making it stick

The last problem we have to tackle is that any variables you type directly into the PMC are transient. As soon as you close Visual Studio, you're going to close that instance of the console and flush your lovingly crafted map variable away.

To get around this little nightmare, you can take advantage of the fact that PowerShell consoles have an associated "profile", which is really just a script that runs every time the shell starts up. 

The PMC has one that is usually a file called `NuGet_profile.ps1` in a folder called `WindowsPowerShell` in your profile's documents folder. e.g. "C:\Users\Bennor\Documents\WindowsPowerShell\NuGet_profile.ps1"

You can find the exact path you need by executing the following command in the PMC:

```powershell
$profile 
```

Easy, huh?

If you don't monkey about with PowerShell very often, this file (and the folder it lives in) probably don't exist, so you may need to create them first.

Once the profile script file is created, fire it up in your favourite text editor. You can do this easily from the PMC with the following command:

```powershell
notepad $profile
```

Once you've got it open, paste in the map variable assignment command from step 1 above, then save and close the file.

**Gotcha** Because this profile script only runs automatically at the time the shell is fired up, you'll have to run it manually to see changes to it take effect. The easy way to do this is just invoke it:

```powershell
Invoke-Expression $profile
```

This will run the script and apply any changes you've made to the variables in it.

## To sum up&hellip;

[summary goes here]

[1]: http://xkcd.com/303/