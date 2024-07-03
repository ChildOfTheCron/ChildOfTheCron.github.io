---
layout: post
title:  "Building Boto3 docs locally for research"
date:   2024-07-01 12:37:33 +0100
categories: security cloud
---

# Building Boto3 docs locally for research

### The background

I've been trying to catch up on fwd:cloudsec talks over the last week. One of the last talks I watched was the "[Get into AWS security research as a n00bcake]([https://www.youtube.com/watch?v=jEFGzLbG1r4](https://www.youtube.com/watch?v=jEFGzLbG1r4))" talk by Daniel Grzelak. It's a great talk and I highly recommend folks check it out if you are new to cloud research. Or in my case, if you want to make sure your not missing tricks because you live under a rock. In the talk Daniel covers the use of the AWS documentation as one of the jumping off points for research. I'm a huge fan of this, as this is normally how I try and go about doing things. 

A side-effect of my day job is I often need to read a lot of AWS (and Azure uurgh) docs. So it seems only natural to make notes of interesting things to investigate later. One point that stood out to me from the talk was compiling a local copy of the documentation so that you could process it more easily. I am a huge fan of grepping for stuff, so I thought this was a great idea!

### The idea

Rather than looking at the AWS documentation itself, I was interested in the Boto3 scripting reference, as at the time I was using boto to run and grab some output for network analysis. Something that I thought was interesting was the DryRun argument that you could pass into various functions in Boto3. **My goal here is to find all the services in AWS Boto3 that accepts the DryRun argument.** This will be useful for some future research I am planning on doing, which I won't go into now. I might write another post once I have done the research and something useful comes from it.

To create this list I thought I’d take the advice from the talk, and compile a local copy of the Boto3 documentation, then grep for it. I thought it might be useful to document my process for others who want to do something similar.

### The method

I started by cloning the boto3 repo available here:

    git clone https://github.com/boto/boto3.git

I then installed the pre-req’s along with sphinx (which is required for Make to build everything).

    sudo apt-get install python3-sphinx
    pip install -r requirements-docs.txt

The next section of the Boto3 repo README shows to switch into the docs directory and pass make to build the html docs.

    cd docs
    make html    

But! Rather than doing that lets open the Makefile and have a poke around.

    ~/REPOS/boto3/docs on develop ● λ: vim Makefile
    
As we can see here, we are able to output the Boto3 info in various forms. 

<div style="text-align: center;">
<br>
<img src="make_opt.png" alt="make_options"/>
<br>
<br>
</div>

I tried json first, but it was a bit overkill for my purposes. I would probably use this if I had some cool way of interpreting fjson. For now though, I just wanna grep for stuff, so I rerun Make and pass “text” as the output format.

    ~/REPOS/boto3/docs on develop ● λ: make text
    
This ran for about ~15 to 20 minutes, and output all the text with a couple of warnings, but nothing that looked super serious. We should now have a build directory with all the documentation we require.

    ~/REPOS/boto3/docs/build/text on develop λ: ls
    guide  index.txt  reference

### The result

Now we can see the output, I looked around and found most of the juicy information I want lives in the following directory. 

    ~/REPOS/boto3/docs/build/text/reference/services

Now that I knew this, I could grep for DryRun and get a list of all services that use it as an argument for at least one of its functions. 

     ~/REPOS/boto3/docs/build/text/reference/services on develop λ: grep -ri 'DryRun' > ../grep_out

<div style="text-align: center;">
<br>
<img src="raw_out_1.png" alt="raw_output"/>
<br>
<br>
</div>

The data output is pretty rough, but this is still a lot faster than searching through documentation by hand! 

With a bit more work, some wrangling and some scripting we can parse the output of the rough grep into something more usable. But I am Le Tired, and the document is pretty short. So a simple regex in vim will do for now. What we end up with is the following list of services that all accept DryRun as an argument.

    migrationhub-config.txt:             DryRun=True|False
    appstream.txt:             dryRun=True|False
    ssm.txt:             DryRun=True|False,
    lambda.txt:             DryRun=True|False,
    network-firewall.txt:             DryRun=True|False
    sms.txt:      * "SMS.Client.exceptions.DryRunOperationException"
    meteringmarketplace.txt:             DryRun=True|False,
    license-manager.txt:             DryRun=True|False
    redshift.txt:             DryRun=True|False,
    mgh.txt:             DryRun=True|False
    clouddirectory.txt:             DryRun=True|False
    ec2.txt:             DryRun=True|False,

Note that for appstream the dryRun starts with a lowercase d which is inconsistent with all other mentions of DryRun in the boto3 docs. We passed the case-insensitive flag in our grep (-i) to catch occurrences such as this.

Looking at the data one thing to note is that 96.8% of all the "DryRun" string appearances seems to be tied to the EC2 service in some way. The other services listed here make less use of it. However these are the services I am interested in looking at. I feel that the less popular something is, the more shenanigans we can do with it. If we disregard the EC2 service the breakdown for number of DryRun string mentions is:

<div style="text-align: center;">
<br>
<img src="dry_run_stat.PNG" alt="pie_charts_with_some_stats"/>
<br>
<br>
</div>


### The missing S3 sync dry-run 
But why is S3 sync dry-run missing? According the the [docs here](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html), s3 sync has a dry run option, and in the data we pulled we can't see any mention of it. Why is this? Well according to some googling, Boto3 doesn't support S3 sync. So it's totally possible that to get a full list of dry-run options, we'd need to refer to the aws CLI documentation instead of the boto3 docs. TIL, I guess!

### The Conclusion
To summarise, I learnt that:

 - EC2 service makes a loooooot of use of the DryRun argument in Boto3
 - Not all the DryRun arguments have consistent casing among the services in Boto3
 - Not all DryRun options are covered (or supported) by Boto3

Is any of this information useful? I have no idea yet. But getting this overview would be difficult unless you are able to manipulate the docs en-mass. Hopefully the article was useful in showing a way to automate some of the mundane slog through AWS documentation. I am hoping to use this method in the future to help quickly analyse data to prove or disprove my theories. Maybe it'll be helpful to you as well!
