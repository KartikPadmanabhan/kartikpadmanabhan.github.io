---
layout: post
section-type: post
title: Calibrating IO for PQ Auto Degree Of Parallelism
category: dba, administration, performance tuning
tags: ['database adminstration', 'oracle']
---
Requirement to enable "parallel_degree_policy = auto" is to run "DBMS_RESOURCE_MANAGER.CALIBRATE_IO". 

CALIBRATE_IO procedure itself impose further requirements like:

* timed_statistics=TRUE
* ASYNCH_IO=TRUE
* Run once in RAC, runs from one instance while all instances are opened.


<pre> <code data-trim class="sql">
set pagesize 200
set linesize 200

SET SERVEROUTPUT ON
DECLARE
  ISPQ varchar2(10);
  ISTS varchar2(10);
  ISAO INTEGER;
  lat  INTEGER;
  iops INTEGER;
  mbps INTEGER;
BEGIN
  EXECUTE IMMEDIATE 'select VALUE from v$parameter where NAME=''parallel_degree_policy''' into ISPQ;
  EXECUTE IMMEDIATE 'select VALUE from v$parameter where NAME=''timed_statistics''' into ISTS;
  EXECUTE IMMEDIATE 'SELECT count(*) FROM V$DATAFILE F,V$IOSTAT_FILE I WHERE  F.FILE#=I.FILE_NO AND  --
                     FILETYPE_NAME=''Data File'' AND    ASYNCH_IO=''ASYNC_ON''' into ISAO;

  dbms_output.put_line('parallel_max_servers = ' || ISPQ);
  dbms_output.put_line('timed_statistics = ' || ISTS);
  dbms_output.put_line('ASYNCH_IO counts = ' || ISAO);

  IF (ISPQ ='auto' and ISTS ='TRUE' and ISAO > 0) THEN
     -- DBMS_RESOURCE_MANAGER.CALIBRATE_IO (<DISKS>, <MAX_LATENCY>, iops, mbps, lat);
     DBMS_RESOURCE_MANAGER.CALIBRATE_IO (2, 10, iops, mbps, lat);
 
     DBMS_OUTPUT.PUT_LINE ('max_iops = ' || iops);
     DBMS_OUTPUT.PUT_LINE ('latency  = ' || lat);
     dbms_output.put_line('max_mbps = ' || mbps);
  ELSE
     dbms_output.put_line('Not eligable for CALIBRATE_IO');
  END IF;
end;
/

</code></pre>

