
# Process analysis with ProM#

Author:  W.Dijkhuis   
Date:  august 2016   

## introduction

This report documents the use of [ProM Lite](http://www.promtools.org/doku.php?id=promlite11) to explore and analyse process behaviour and to find the process structure.  
The analysis is a mini project that was suggested by the authors of the TUe/FutureLearn MOOC [Introduction to Process Mining with ProM](https://www.futurelearn.com/courses/process-mining). 

The data consist of an event log,  recording the processing of traffic fines issued by the Italian police.  
Beyond that, no information about the process was given.   
All other information had to be deduced from the data in the event-log.

.

## 1 - data acquisition

The data were collected by the authors of the MOOC in a file Road_Traffic_Fine_Management_Process.xes.gz  
The data were already correctly formatted to be used in ProM.  
The file could be downloaded from the internet.  

.
 
## 2 -  data exploration

The the ProM log visualizer gives the following overview

#### figure 1 - log visualizer results

![figure 1 - visualizer results](.\fig\\fig01_log_visualizer.jpg) 

Here we read:

- there are 150,370 traces (i.e.fines that are processed)
- there are 11 event classes (i.e. different steps in the processing of fines)
- there are between 2 and 20 events processed per trace
- there are between 2 and 10 different event classes per occurring in traces.
- some event classes reoccur within a trace (so there are loops in the process) 

#### figure 2a - dotted chart: event name by time (colour represents index)

![figure 2a - dotted chart: event name by time](.\fig\\fig02a_dotted_chart_eventname_by_time.jpg) 

Here we read:

- The data were collected between 2000 and 2013 (so we have about fourteen years of data).
- There are 11 event types, 
provisionally reordered and grouped these are: 
	- a: create fine
	- b: send fine 
	- c: insert fine notification
	- d: add penalty (optional, occurs for late payment?)
	- e: payment
	- = = = = =
	- f: send appeal to prefecture
	- g: insert date appeal to prefecture 
	- h: receive results appeal to prefecture
	- i: notify results appeal to offender
	- j: appeal to judge
	- = = = =
	- k: send for credit collection
- the events a, b and c almost always occur (their dotted lines are solid) 
- the lines a, b, c lines have constant colour. The colour represents the index of the event in the trace (is it the first, second, third, etc. event in the trace). So the a, b and c are processed in a fixed sequence (probably a->b->c)
- The line for event e -payment- is solid, so most fines are paid in the end. The line for k "send for credit collection" only contain a few dots, this supports the idea of frequent payment. 
- The e line is multicoloured, so payment has not a fixed index (can occur as the fourth, the fifth, the sixth etc. step in the process).
- probably e is the last step in most traces.
- the d line for "add penalty" is solid also; that would means that this occurs frequently.       

Figure 2a was coloured by index position.   
If we colour by event name we get figure 2b.   
Here we can see which colour will represents which event in figure 2c .

#### figure 2b - dotted chart: colour per event name (colour represents event name)

![figure 2b - dotted chart: colour per event name](.\fig\\fig02b_dotted_chart_colour_per_eventname.jpg) 

Now we can create figure 2c.

#### figure 2c - dotted chart: index by time (colour represents event name)

![figure 2c - dotted chart: index by time ](.\fig\fig02c_dotted_chart_index_by_time.jpg) 

Here we read:

- The hypnotised standard sequence a->b->c-(d)->e indeed occurs in that order (i.e. in index positions 0,1,2,3 and 4).
- Some fines are paid early (traces a->e and a->b->e also occur)
- legal appeal events occur not that often, when they occur at index 4, 6, 7 or 8.

Probably we can distinguish two types of normal cases  
1) cases without appeal (containing only events a,b,c,d,e and k)   
2) cases with appeal (also containing f,g,h,i and j)  

Note that figure 2c has strange features.
- the maximum event index is 20 (figure 1 showed that the maximum number of events per case was 20, so that is consistent). However figure 2c has a maximum index of 12. No cases with index 13 or higher occur. This can only be explained by assuming that the figures do not show the whole data set, but only a sample.
- there are two cases that have index 12 but have no index 11.      

Figure 2d shows how cases are processed in time. On the horizontal axis the trace index is displayed (first trace in the file, second third etc.)

------
     
The explore event log (trace variants) visualization option provides and easy way to find typical way to go trough the process. Figure 3 shows the five most common trace types.

#### figure 3 -  the five most common traces

![figure 3 -  five most common traces](.\fig\\fig03_top_5_traces.jpg) 

Here we read:

- the five most common traces cover over 90% of cases.
- None of these five involve going to court.
- Only about 40% of cases end with payment
- Alarmingly 50% of cases end in non payment  
(are these cases still in progress? or "lost" in the system?)
- the hypothesised order a->b->c->(d)->e sequence is clearly visible.
- it is possible to pay in instalments.

-----

The "mine for fuzzy model" plug-in can produce a simple preliminary process model.  
After the (laborious!) intervention of changing the layout by hand the result can look as in figure 4. 

#### Figure 4a - a preliminary process model

![figure 4a - preliminary process model](.\fig\\fig04_fuzzy_mining_process.jpg) 

The model has several remarkable features.

To start with it is not clear what the possible end-states of the process are. One possible interpretation is:

- The process ends when the fine is paid in one or more instalments.   
However even after payment, a penalty can be added  (if the fine is not fully paid on time?).     
So one way the process can end is paying fully on time.   
From figure 3 we know that about 30% of cases end like this.
- The last step in the process can also be the "Send for credit collection complete".   
Note that for the offender the case has not ended yet, but the police has handed the "problem" over to the credit collecting authority, so the police has done its work, for them the case is closed.  
From figure 3 we know that about 38% of cases end like this. 
- A case can also end after appeal to the Prefecture (if the appeal is granted and the offender notified).   
If the appeal is not granted the offender can appeal to a Judge.  
It is not clear what happens when the first appeal is not granted and the offender does not appeal to a judge.
- Finally a case can end when a judge grants an appeal. 

Secondly, from figure 3 we know that about 10% of cases end in "add penalty" followed by "payment".  
The process model in figure 4 does not allow for this possibility at all.  
(maybe the arc between "payment" and "add penalty" should point the other way?).

Thirdly, from figure 3 we know that almost 14% of cases consists of "create fine" followed by "send fine" and nothing else. The logical next step "insert fine notification" is missing. One possible explanation would be that the notification is sent to an address where the offender is unknown. The notification can not be delivered, the fine is non-collectable, the process ends here. If that were the case there should be an activity "insert fine is not notable". Maybe the designers of the event log forgot to record this activity.   

**Note** the event-log is not the only possible source of information.  
Subject matter experts can probably solve most problems we encountered.

-----

The "mine Petri net with inductive miner" option can produce an somewhat more detailed preliminary model.

#### Figure 4b - a preliminary Petri-net process model

![figure 4b - a preliminary Petri-net process model](.\fig\\fig04b_inductive_miner_petrinet.jpg) 

The model has several remarkable features.

- First all activities except "create fine" and "add penalty" can be bypassed/skipped (there is a black box directly parallel to those activity). 
- Second It is even possible that the whole process consist of only the first step "create fine" (follow the "uppermost" path trough the graph).
- Third there are no loops in figure 4b. Even the repeated payment loop that we can see in figure 3 can not be produced by the Petri net from figure 4b. According to figure 4b one can do a payment only once per trace.
- Fourth in figure 4b there is no direct connection between the "add penalty" and the "payment" activity. From figure 3 we know that about 10% of cases end that way.

----

A very mysterious pattern occurs when we make a  dotted chart that shows how events occur in time.  

#### figure 5 - dotted chart: trace processing in time (colour represents event name)

![figure 5 - dotted chart: trace processing in time](.\fig\\fig05_dotted_chart_tracesstarttime_by_eventime.jpg) 
 
When cases are entered when they are processed in reality we would expect a line around the main diagonal. 
This expected band around the main diagonal is clearly visible e here., but there is much much more.

The dark green vertical stripes are credit collection events, that are preformed once each year.

Many event time stamps are years after the trace  start time. This might be explained when large part of the data were entered into the system retrospectively. However there are also many traces that are entered before they occur. 

Something very funny is going on here, about half of the data are positioned on strange and even impossible positions in figure 5.      

.

## 3 - data filtering

Figure 6 gives a hint which cases to select to get data that do behave orderly (not crazy as in figure 5).

#### Figure 6 - trace concept name by time stamp  (colour represents event name)

![figure 4b - 6 - trace concept name by time stamp](.\fig\\fig06_traceconcept_name_by_time.jpg) 

If we select traces with concept name between S130,000 and S150,000 then we have a data set that develops orderly along the main diagonal. It will be data of 2008, 2009, and 2010, so we may assume that most cases are closed (the data collection continuous until the middle 2013, so there are 3.5 years to close cases from 2010).

The Filter Log on Trace Attributes plug-in can filter on concept name. Select all traces with name between s130,000 and s150,000 (and for reuse, export the result to disk, export as .xes file).

If we show figure 5 for this subset we get:

#### figure 5b - dotted chart for subset: trace processing in time (colour represents event name)

![figure 5 - dotted chart for subset: trace processing in time](.\fig\fig05b_dotted_chart_tracesstarttime_by_eventime_for_subset.jpg) 

The shown behaviour makes sense. Traces start with a "create fine" event (light green)  are often followed by a "payment"  event (yellow) and then by a insert fine (dark blue) etc. Note that there is little activity after 2012, so we may assume most that most cases are closed.

According to the log visualizer there are 10441 traces and 36666 events in our selection. This is ample data to work with. 


Repeating figure 3 for the subset gives:

#### figure 3b - the five most common subset traces 

![figure 3b - the five most common subset traces](.\fig\\fig03b_top_5_traces_for_subset.jpg) 
           
These five patterns cover about 95% of cases

**Note**  
- the top 4 traces in figures 3 and 3a are the same (slightly repositioned).  
- the last trace in 3a ends in send appeal to prefecture. According to common sense this can not be an end event of a trace (the prefecture has yet to do its work). Probably these cases (2.15%!) are still open (Italian justice is famous for working very slow).

-----

Legal ends of traces might be "payment" and "send to credit collection". In exceptional cases a trace might end with "notify results appeal to offender" or "appeal to Judge complete" (see figure 4).

The "Filter Log using Simple Heuristics" can be used to remove all traces that have no valid end, i.e. use:

- valid begin: "create fine".
- valid end: "payment", "send to credit collection", "notify results appeal to offender" or "appeal to Judge complete"
- keep all events.
 
According to the log visualizer there are 8534 traces and 31737 events in our selection. This is ample data to work with. 

**Note** that  10441 - 8534 = 1907 traces (18%) were eliminated. That would mean that about 18% of cases are "lost" in the system or still in progress. From figure 3b we know that about 15% of the traces in our subset consist of "create fine" followed by "send fine", the trace ends there, possibly because the offender is unknown at the address were the fine is sent to.

Repeating figure 3 for the new subset gives:

#### figure 3c - the five most common valid-end subset traces. 
      
![figure 3c - the five most common valid-end subset traces](.\fig\\fig03c_top_5_traces_for_valid_ends_subset.jpg)

Futher more there are 6 traces (in 5 variants) that end in "appeal to Judge" and 2 cases end in "notify results appeal to offender". All 38 other cases where "notify results appeal to offender" does occur end in one of the three other valid ends. **note** only 8 traces out of 8534 end with  a"appeal to Judge" or "notify results appeal to offender", we could remove these without significant loss.

## 4 - process discovery on filtered data

First lets make the equivalent of figure 4a for our filtered data (i.e. use "mine for fuzzy model" plug-in).

#### Figure 4c - a fuzzy process model on filtered data

![figure Figure 4c - a fuzzy process model on filtered data](.\fig\\fig04c_fuzzy_mining_process_on_valid_ends_subset.jpg) 

This model is largely correct (i.e. in conformance with that which we know) and plausible.  
- The two most common trace patterns from figure 3c are clearly visible.
- the Prefecture appeal procedure has been found.
- there is a payment loop (and a transition to "send for credit collection").

The most troubling features are:
- three of the patterns of figure 3c (12% of cases) can not be produced by the model.
- there is no direct link from "add penalty" to "payment" (figure 3c shows that at least 11.5% of traces use this transition, the lack of that transition is a big problem).
- There is a direct link from "create fine" to "appeal to Judge". Non of the traces in our data has this transition. "Appeal to Judge" can only occur after the verdict of the Prefecture event "Notify results Appeal to offender". The last transition is missing in the model.

----


 
 

       