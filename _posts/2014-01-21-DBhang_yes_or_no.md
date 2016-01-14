---
layout: post
section-type: post
title: Quick and Dirty Commands For Analyzing Database Hang
category: dba, administration, performance tuning
tags: ['database adminstration', 'hang analysis', 'oracle']
---

Quick and dirty tutorials on database lock triaging

## Do I have A DB hang? NO

Its a process hang and can connect / as sysdba with sqlplus
Run the following SQL to identify a blocker that is holding resources for more than 10 minutes (if exists),


<pre> <code data-trim class="sql">

	SET PAGESIZE 60 COLSEP '|' NUMWIDTH 8 LINESIZE 132 VERIFY OFF FEEDBACK OFF
	set linesize 200
	col IN_WAIT_SECS form 999999
	col OSID form a10
	col BLOCKER_OSID form a12
	col WAIT_EVENT_TEXT form a30
	
	SELECT IN_WAIT_SECS, INSTANCE, OSID, BLOCKER_INSTANCE, BLOCKER_OSID, WAIT_EVENT_TEXT
	FROM   v$wait_chains
	WHERE  IN_WAIT_SECS > 600 AND
	BLOCKER_OSID IS NOT NULL
	ORDER BY BLOCKER_INSTANCE, OSID;

</code></pre>

Example output from a hung process:


<pre> <code data-trim class="bash">

	IN_WAIT_SECS|INSTANCE|OSID      |BLOCKER_INSTANCE|BLOCKER_OSID|WAIT_EVENT_TEXT
	------------|--------|----------|----------------|------------|------------------------------
	       56159|       1|19783     |               5|4360        |library cache lock
	       51832|       1|20350     |               5|4360        |library cache lock
	       56149|       5|4360      |               5|32352       |PX Deq: Parse Reply

</code></pre>

Note. ospid 32352 on inst 5 is the potential blocker, thus killing it should relief the system.

### Collect sufficient info. (all will be in *diag*trc):

	#### HANGANALYZE: 

	<pre> <code data-trim class="sql">	
		oradebug -g all hanganalyze 4
	</code></pre>
	
		* "Chain 1" is critical
		* Chains will have the short call stack

	#### SYSTEMSTATE: 

	<pre> <code data-trim class="sql">	
		oradebug -g all systemstate 10
	</code></pre>
	
## Do I have a DB hang? YES

Can't connect with sqlplus . as sysdba (it too hangs)

The "oradebug hanganalyze" command is not supported in prelim (sqlplus -prelim ..).

If you want prelim mode hang analysis dumps you have to do the following.

### Hang analysis in prelim mode:

#### Single Instance (non-rac):

	<pre> <code data-trim class="sql">	
		sqlplus /nolog  
		set _prelim on  
		connect / as sysdba  
		oradebug setorapname diag  
		oradebug dump hanganalyze 1 
		oradebug tracefile_name
	</code></pre>

#### RAC:

	<pre> <code data-trim class="sql">	
		sqlplus /nolog  
		set _prelim on  
		connect / as sysdba  
		oradebug setorapname reco 
		oradebug dump hanganalyze_global 1  
		oradebug setorapname diag  
		oradebug tracefile_name
	</code></pre>

### NOTES:

* for RAC, RECO is specified and not DIAG. RECO will message DIAG
* In both cases the hang analysis output can be found in the local DIAG trace file.
