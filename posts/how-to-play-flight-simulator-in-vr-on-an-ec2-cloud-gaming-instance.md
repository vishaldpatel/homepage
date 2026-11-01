---
title: "How to play Flight Simulator in VR on an EC2 Cloud gaming instance"
date: 2026-03-24T19:30:43Z
updated: 2026-04-30T20:34:17Z
slug: how-to-play-flight-simulator-in-vr-on-an-ec2-cloud-gaming-instance
published: true
---

# How to play Flight Simulator in VR on an EC2 Cloud gaming instance

## 3rd Update

You might get tired of having to log into AWS Console every time you want to start / stop / find the IP address for your gaming instance. At least, I did. So, I installed [the AWS Command Line Interface](https://aws.amazon.com/cli/). After getting it setup (creating a new IAM role and restricted policy limited to just EC2), which didn't take long, I'm able to start and stop my one EC2 instance from the command line, and find out the IP address for my instance when it is started using three commands.

To list your instances (instance IDs and IP addresses):

```sh
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId, PublicIpAddress]' --output table
```

To start your instance:

```sh
aws ec2 start-instances --instance-ids i-your-instance-id
```

To stop your instance:

```sh
aws ec2 stop-instances --instance-ids i-your-instance-id
```

Being paranoid that I am, I run the command to list instances after stopping my instance a couple of times. When the instance is running, it is assigned an IP address. When it is stopped, the IP address is unassigned.

## 2nd Update

Suggestions to dramatically improve VR streaming performance can be found in [this Reddit thread](https://www.reddit.com/r/OculusQuest/comments/1ak4kps/what_settings_do_you_all_use_in_virtual_desktop/) and some Flight Simulator 2024 specific settings can be found in [this YouTube video](https://www.youtube.com/watch?v=I9S00G8Ovk8).

## Update

I created a more concise set of steps for the confident: [see it here in Powershell script format](https://gist.github.com/vishaldpatel/6efe63bbc4f80182c434a628c1b24d97).

## Original post

Not so long ago, I started playing Microsoft's Flight Simulator 2020 and Flight Simulator 2024. I've played them using NVidia's GeForce Now service as well as Shadow PC. Both services are excellent and I highly recommend them!

Then, I realized that the games would be incredible to play in VR! So, I got a Meta Quest 3 headset.

Unfortunately, Shadow PC's most powerful tier isn't powerful enough to play in VR, and nVidia's GeforceNow service does not yet have support for full VR. They promise that they are working on it. In the meantime, I set up an Amazon ec2 instance.

Follow these tutorials:

- [Step by Step instructions on setting up an ec2 instance](https://aws.amazon.com/blogs/gametech/game-on-demand-unlocking-cost-efficient-cloud-gaming-with-amazon-ec2s-pay-as-you-go-model-using-steam/)
- [Another set of instructions for doing the same](https://aws.amazon.com/blogs/compute/use-amazon-ec2-for-cost-efficient-cloud-gaming-with-pay-as-you-go-pricing/)

## A few notes

- AWS's ec2 service lets you basically rent a cloud based computer by the hour. They have many types of computers. We're interested in the computers that let us play 3D games - these are generally their computers ("instances") that start with the letter "G", such as "G4.2x.large" or "G7e.4x.large" etc..
- If you are new to AWS / ec2, you will need to request access to ec2 instances (computers) that can do graphics heavy work. You will have to go through their support portal. There is a form for requesting an increase to your vCPU / GPU quota for G class instances. It is set to 1 or 0 by default. I highly recommend requesting 8 CPUS to begin with. Be sure to explain your use-case. It will take a few days for them to approve it.
- For now, I have opted for a more powerful **G7e.2xlarge** instance rather than the "G4.." and "G5.." instances because I want to play my a very heavy Flight Simulator 2024 in full VR.
- While you don't get charged unless your instance is running (I shut mine down every time I'm done playing), you do get billed for hard disk provisioning! I suggest being more conservative with how much Disk space you provision. The default is something very low like 25 Gigs, which is just big enough for Windows 11. I bumped mine up to 120 GB.

## There are a few things you'll need to install immediately on your newly created instance

- [Download and Install Amazon DCV server for windows x64](https://www.amazondcv.com/)
- [Install AWS tools and enable script execution](https://docs.aws.amazon.com/powershell/v5/userguide/ps-installing-awstools.html)
- [nVidia gaming drivers](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nvidia-gaming-driver.html)
- [Virtual Desktop Streamer App for Windows](https://www.vrdesktop.net/)

Virtual Desktop is the best solution with the least amount of fuss that I have found for streaming games onto a Meta Quest 3. You will have to pay - it cost me around $25 USD. The Streamer is free, but the Quest 3 client is what you pay for.

You will likely want to use a Playstation or XBox controller. The only way to do this is to log into your EC2 instance using either a Windows DCV client of from Chrome on **Port 8443** ([more information on that here](https://docs.aws.amazon.com/dcv/latest/adminguide/enable-gamepad.html)). So, you'd go to `https://your.ec2.ip:8443` (don't forget the https!). Ignore the browser's security warning and connect.

Due to AWS's limitations, the way you play VR games is the following:

1. Connect your XBox controller to your regular computer / laptop, and log into the EC2 instance using Chrome if you are on a Mac (or possibly Linux) or the Windows DCV Client.
2. Once you login, press a button on the controller for Windows to recognize it. It will make a ding sound. You may still have to install drivers - [see this article](https://docs.aws.amazon.com/dcv/latest/adminguide/enable-gamepad.html).
3. In order to reduce bandwith use from your DCV client, switch its settings to use DCV in 800x600 resolution. Don't worry - this won't affect your VR experience.
4. Start the game.
5. Put on your headset, connect to your EC2 instance using Virtual Desktop. Switch to VR mode for the game - in the case of Flight Simulator, you go through the game's settings.
6. Go Fly!

There's still a bunch of tweaking that can be done to improve the overall experience:

- Make sure the ec2 instance is as close to you as possible to reduce lag.
- Tweak the Virtual Desktop settings to improve streaming quality.
- Tweak the nVidia driver settings to improve graphics performance and quality.
- Tweak game specific settings to improve graphics and performance.
