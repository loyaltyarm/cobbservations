+++
date = "2013-07-13T17:53:26-05:00"
title = "Using Splunk for McAfee EEMac/Filevault 2 Comparison - Part 2"
draft = false

+++

[In Part 1](https://cobbservations.com/blog/using-splunk-for-mcafee-eemac/filevault-2-comparison--part-1/), we looked at setting up Disk I/O monitoring using Splunk. In Part 2, we will look into formatting our search in order to get the most of our data.

To place this post in the context of its title, I'm moving away from using the machine that was shown in Part 1 as connected to Splunk (`loyaltyarms-mac.local`). That was merely to show you how to get your data into Splunk. Now, we will look at pulling search data for Disk I/O from 2 sources added to my Splunk environment for this test. Those machines are `test-fv2` (shown as `test-fv2.local`) and `test-mcafee` (shown as `test-mcafee.local`). Let's get started! 👍

The first machine we will setup search for will be the FileVault test MBA. In searching for data from a specific host, we will want to refine our search to only show us data from that host specifically. Therefore, the first thing we will do is proceed to the _Search_ app in Splunk, and click the _Summary_ or _Search_ menus in the toolbar. Next, we will enter the host information into the search bar:

`host="test-fv2.local`

![forwarder package](/img/splunk-host.png)

As we enter this information, Splunk’s search bar will pop down a very useful context menu for assisting with search terms and syntax. There are even links to documentation to view the best way to format a search. Go ahead and enter the text above into your search bar and hit _Enter_. (Even easier, you may click on the host’s name in the bottom right of the _Summary_ screen to display results immediately for that host.) After you press _Enter_ (and if Splunk is indexing events from your host as typed), the screen changes to a view displaying a timeline of results across the top, and a table of the results down below (shown above). Along the left column is a list of fields that Splunk was able to recognize just from indexing the forwarded results. You can add these fields to this and other searches as additional fields. It is worth mentioning that if you have additional sourcetypes for a given host, you may want to restrict the results further. To do this, simply enter:

`sourcetype="<x>"`

...where `<x>` is replaced with the sourcetype of your Disk I/O data. In my case, the sourcetype is named _Apple Disk I/O_.  Now that we know for sure that our data is restricted to the machine and type that we want, we can proceed with narrowing our search for the four fields listed in Part 1. Splunk has excellent documentation on searching, since that is the primary way of getting custom reporting out of the software. You can get started with Search documentation [here](http://docs.splunk.com/Documentation/Splunk/latest/Search/Whatsinthismanual). [This is also another good starting place.](http://docs.splunk.com/Documentation/Splunk/latest/SearchReference/WhatsInThisManual) This first thing we will do is separate our initial search requirements from our refinements with a pipe `|` character to give Splunk multiple search commands. In the search box where your search for host and sourcetype exists, use a tailing space and a pipe and the multikv command to extract field-values from table-formatted disk i/o results we are now showing. Our search contain the same text as below:

```
sourcetype="Apple Disk I/O" host="test-fv2.local" | multikv fields Device, rReq_PS, wReq_PS, rKB_PS, wKB_PS |
```

![forwarder package](/img/splunk-sourcetype.png)

What did we do here? We used the `multikv` command to set the fields we want to extract data from. These were our data headings from the previously-shown results: 

```
Device
rReq_PS
wReq_PS
rKB_PS
wKB_PS
```

Since we are now showing our data in table-form without the headers, we now want to create some kind of visualization or reporting pattern that allows us to examine certain fields over time. In Part 1, we discussed that `rReq_PS` and `wReq_PS` represent the values for disk Operations/sec for Read and Write, respectively. For now, let’s go ahead and create a [timechart](http://docs.splunk.com/Documentation/Splunk/6.6.0/SearchReference/Timechart) for those two fields, and sort them by Device, using the full search string below:

```
sourcetype="Apple Disk I/O" host="test-fv2.local" | multikv fields Device, rReq_PS, wReq_PS, rKB_PS, wKB_PS | timechart avg(rReq_PS) avg(wReq_PS) by Device
```

![forwarder package](/img/splunk-sourcetype2.png)

Now our data is shown in a somewhat-simplified table. As a side note, Splunk has a number of charting options available as search commands, just follow the links from earlier to view the Splunk search manuals and documentation.

At this point you can click on the _Results Chart_ icon in the top left of the table view area to sort this data in a visualization of your choosing. However, if we examine this table closely, we notice that our headings are still represented in their original syntax as returned from the `iostat.sh` script we initiated earlier. Wouldn’t it be better to rename these fields for ease of reporting? Our director may end up with this report, and we want he/she to understand what he/she are reading here, so let’s add to our search to make that a little easier to understand. Our search now extends one level further, using the `rename` command.

```
sourcetype="Apple Disk I/O" host="test-fv2.local" | multikv fields Device, rReq_PS, wReq_PS, rKB_PS, wKB_PS | timechart avg(rReq_PS) avg(wReq_PS) by Device | rename "avg(rReq_PS): disk0" AS "Operations (Read): disk0", "avg(wReq_PS): disk0" AS "Operations (Write): disk0"
```

Now our columns are renamed accurately, but our search bar is getting crowded. That's okay, because at this point we are finished with adding specific search criteria. If additional disks were found in your reporting, you can easily reformat those columns using the same string we did above, just change the column name between `rename` and `AS` to match your rogue column (as many times as you need to).

![forwarder package](/img/splunk-sourcetype3.png)

This gives us a good overview of how to search and an understanding of how we can pull information from Splunk. For the test comparison we are doing here, we will be formatting our search to include multiple hosts, so as to show all of the results of the different reporting types (_Operations_ vs. _KB_) across both machines on the same visualization. For this we will utilize _Report Builder_ [(Part 3)]() after we have tweaked our search a bit. We will also be adding a custom time to our search in order to be able to look at different points in the encryption process and compare. We will go ahead and add custom time now, so that you can play with your results in the meantime to see if they differ for your own testing.

Once we have the search formatted as above, we can navigate to the right of the search bar and select the green button with time indication text displaying on it. Clicking on the green time button reveals some pre-configured searches, as well as the ability to configure a custom time range to search for results based on our search syntax. Let’s select custom time here. When clicked, we are met with a custom time selection box, shown here:

![forwarder package](/img/splunk-customtime.png)

Select the period that you would like to monitor. For me, I want to monitor the specific 24 hour period in which the FV2 Mac was idle, and then encryption began. *The results to be shown are fairly generic, but for [Part 3](), I will be posting the exact times and dates as reported in order to display accurately compared results. After the search has been formatted, you can see that the results are again displayed in table format, and can be configured for Visualization reporting based on that time period using the _Results Chart_ button from earlier.

![forwarder package](/img/splunk-tableview.png)

Now that you know how to add custom search times, see what you can come up with. In [Part 3](), we will use _Report Builder_ to build a report based on a specific complex search of the 2 machines, and report on their disk i/o performance during 4 key times in the encryption process. Stay tuned! 📻
