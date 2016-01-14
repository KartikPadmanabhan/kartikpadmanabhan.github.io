---
layout: post
section-type: post
title: Quick and Dirty Commands For Analyzing Database Hang
category: dba, administration, performance tuning
tags: ['database adminstration', 'hang analysis', 'oracle']
---

Quick and dirty tutorials on database lock triaging

## Dump system state and hanganalyze (from one instance) 


<pre> <code data-trim class="yaml">

   oradebug -g all dump systemstate 10
   oradebug -g all hanganalyze 4

</code></pre>

## Find process with eXclusive request (on each instance)

<pre> <code data-trim class="yaml">

	grep 'RequestMode=X' *_diag_*.trc

</code></pre>

## Who is blocking this Mode=x

####  view the trace file

<pre> <code data-trim class="yaml">

	 e.g. /RequestMode=X

</code></pre>

#### Search backward for ospid [from current position found in (b)]

<pre> <code data-trim class="yaml">

	e.g. ?ospid
       
            owner: 0x11d62cea0 - ospid: 27454
</code></pre>

#### Search HANG dump 

<pre> <code data-trim class="bash">

		e.g. 
           vi *a_diag_*.trc  (instance where oradebug is used from)
           /^HANG
           /os id: 27454

          -------------------------------------------------------------------------------
          Chain 8:
          -------------------------------------------------------------------------------
              Oracle session identified by:
              {
                          instance: 8 
                             os id: 27454
          ..    }
              is waiting for 'library cache lock' with wait info:
              {
          ..
              }
              and is blocked by
           => Oracle session identified by:
              {
                          instance: 4 
                             os id: 11275
          ..
              }
              which is waiting for 'enq: CN - race with reg' with wait info:
              {
                                p1: 'name|mode'=0x434e0006
                                p2: 'reg id'=0x1
                                p3: '0'=0x10004
                      time in wait: 1009 min 43 sec
                     timeout after: never
                           wait id: 1057
                          blocking: 1 session
                       current sql: SELECT customer_trx_line_id FROM ra_customer_trx_lines 
                                    WHERE customer_trx_id = :customer_trx_id AND link_to_cust_trx_line_id is NULL
                       short stack: ksedsts()+461<-ksdxfstk()+32<-ksdxcb()+1876<-sspuser()+112<-__sighandler()<-
                                    semtimedop()+10<-skgpwwait()+160<-ksliwat()+1845<-kslwaitctx()+163<-
                                    kjusuc()+3611<-ksipgetctxi()+1759<-ksqcmi()+24148<-ksqgtlctx()+3866<-
                                    ksqgelctx()+561<-ktcnGetRegEnqueue()+50<-ktcnGetPredicateLock()+23<-
                                    ktcnqQctxAcquirePredicateLocks()+216<-ktcnPublishRegistration()+442<-
                                    ktcnRegisterQuery1()+373<-ktcnRegisterQuery()+875<-ktcncRegisterQuery()+870<-
                                    kpoqqreg()+1043<-selexe0()+1358<-opiexe()+16431<-kpoal8()+2246<-opiodr()+1149
                      wait history:
                        * time between current wait and wait #1: 0.001973 sec
                        1.       event: 'DFS lock handle'
                           time waited: 0.003367 sec
                     wait id: 1056            p1: 'type|mode'=0x49560005
                                              p2: 'id1'=0x53594e43
                                              p3: 'id2'=0x13
                        * time between wait #1 and #2: 0.000101 sec
          ..
    }

</code></pre>

#### Identify problem function

	The first kernel none-service function is ktcnGetRegEnqueue() in example above.
