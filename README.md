~~~~~~~~~~~~~~~~

//***FILE 185 contains tools to help you control the TSO auth       *   FILE 185
//*           tables which APF-authorize TSO commands and programs  *   FILE 185
//*           to your TSO session.  Between this file, and File     *   FILE 185
//*           797, you have a very nice set of tools which will     *   FILE 185
//*           help you in this matter, both in altering the         *   FILE 185
//*           "common storage" copies of the auth tables created    *   FILE 185
//*           by PARMLIB (members IKJTSOxx), and in altering the    *   FILE 185
//*           individual TSO session's auth tables pointed to by    *   FILE 185
//*           the LWA (Logon Work Area).                            *   FILE 185
//*                                                                 *   FILE 185
//*           ASUB deals with the "common storage" tables. (CSA)    *   FILE 185
//*           TSUB deals with the "LWA pointed to" tables. (User)   *   FILE 185
//*                                                                 *   FILE 185
//*           File 185 also contains source code (member ASMTABLS)  *   FILE 185
//*           and a load library (member LOADLIB) to help you make  *   FILE 185
//*           a greatly expanded IKJTABLS load module to authorize  *   FILE 185
//*           programs and commands under TSO.  When run in an APF  *   FILE 185
//*           authorized STEPLIB in a TSO session, this IKJTABLS    *   FILE 185
//*           load module will override IKJTSOxx from PARMLIB and   *   FILE 185
//*           will also override the copies of IKJTEFE2, IKJEFTE8,  *   FILE 185
//*           IKJEFTAP, and IKJEFTNS that are in SYS1.LPALIB.       *   FILE 185
//*           The load module IKJTABLS should be copied into        *   FILE 185
//*           an APF-authorized STEPLIB with all its aliases:       *   FILE 185
//*           IKJEFTE2, IKJEFTE8, IKJEFTAP, IKJEFTNS.               *   FILE 185
//*                                                                 *   FILE 185
//*           The source code (member ASMTABLS) was created with    *   FILE 185
//*           the aid of the vendor product STARTOOL FDM from       *   FILE 185
//*           Serena, Inc.  I have included a free method of        *   FILE 185
//*           generating a nearly equivalent disassembly of your    *   FILE 185
//*           current IKJTABLS load module, using only free tools.  *   FILE 185
//*           This is completely self-contained in XMIT format,     *   FILE 185
//*           as pds member FILE234I in this file.  Member $$$$READ *   FILE 185
//*           in that pds, and member $$NOTE2 in this pds, will     *   FILE 185
//*           explain what to do, if you need to generate your      *   FILE 185
//*           own assembly JCL for IKJTABLS.                        *   FILE 185
//*                                                                 *   FILE 185
//*           The load library is included in this file, in member  *   FILE 185
//*           LOADLIB, in TSO XMIT format.  Just do a TSO RECEIVE   *   FILE 185
//*           INDS(this.pds(LOADLIB)) to create the load library    *   FILE 185
//*           on your system.  There are load modules for other     *   FILE 185
//*           "TSO auth table" tools in this load library as well:  *   FILE 185
//*                                                                 *   FILE 185
//*           ADIS, ASUB, LLWA, LSLT, LWATEDIT, LWATMGR, STEPLIB,   *   FILE 185
//*           and TSUB.                                             *   FILE 185
//*                                                                 *   FILE 185
//*           If you want to authorize everything that everybody    *   FILE 185
//*           else has, you have to copy (zap) all the names        *   FILE 185
//*           from your IKJTSOxx PARMLIB member into this load      *   FILE 185
//*           module, or else you might find that you've lost       *   FILE 185
//*           authorization of some programs and/or commands.       *   FILE 185
//*           I have tried to include all the commands that I       *   FILE 185
//*           use, and all the names that I could find on the       *   FILE 185
//*           live systems, but you may most likely need some       *   FILE 185
//*           more.  To quickly obtain a list of programs in your   *   FILE 185
//*           session's auth tables see the LWATMGR and LWATEDIT    *   FILE 185
//*           tools in CBT File 797.  Also you can run TSUB E2D,    *   FILE 185
//*           TSUB E8D, TSUB APD, TSUB NSD, to display your         *   FILE 185
//*           TSO session's current lists.  Use ASUB E2D, etc.      *   FILE 185
//*           to display the PARMLIB-generated current lists.       *   FILE 185
//*                                                                 *   FILE 185
//*           Also refer to CBT Tape File 797 for extra help in     *   FILE 185
//*           this area.  File 797 has tools to manipulate the      *   FILE 185
//*           TSO auth tables in each userid's session, or to       *   FILE 185
//*           LOAD COMPLETELY NEW TABLES for a user's session.      *   FILE 185
//*           File 185 concentrates on tools to CHANGE the          *   FILE 185
//*           EXISTING TSO auth tables either in common storage,    *   FILE 185
//*           or in your own session.  (ASUB and TSUB               *   FILE 185
//*           respectively.)                                        *   FILE 185
//*                                                                 *   FILE 185
//*           Please read members $EXPLAIN and $$NOTE1 carefully.   *   FILE 185
//*                                                                 *   FILE 185
//*           Use the new ADIS command (Auth table DISplay)         *   FILE 185
//*           command to display what authorized commands and       *   FILE 185
//*           programs everybody else has.  ADIS will generate      *   FILE 185
//*           a "copyable list" of these commands, whether it be    *   FILE 185
//*           from the "live" IKJEFTE2, or IKJEFTE8, or IKJEFTAP    *   FILE 185
//*           tables, or even from the IKJEFTNS table.              *   FILE 185
//*                                                                 *   FILE 185
//*           Members ADIS (source for a TSO command to display     *   FILE 185
//*           the active "auth tables" generated from a PARMLIB     *   FILE 185
//*           UPDATE(xx) command, a SET IKJTSO=xx command, or       *   FILE 185
//*           an IPL, and ADIS$ (JCL to assemble ADIS) have been    *   FILE 185
//*           added to this pds.  And the load module for ADIS      *   FILE 185
//*           has been added to the LOADLIB member.                 *   FILE 185
//*                                                                 *   FILE 185
//*           ADIS will display the common storage TSO/E "auth      *   FILE 185
//*           tables" generated from the PARMLIB member IKJTSOxx.   *   FILE 185
//*           The output of the ADIS command can be captured        *   FILE 185
//*           using SYSOUTTRAP tools, since it is generated by      *   FILE 185
//*           the TSO PUTLINE service.  Try ADIS.  You'll like      *   FILE 185
//*           it.  ADIS is read-only, and does not need APF         *   FILE 185
//*           authorization.                                        *   FILE 185
//*                                                                 *   FILE 185
//*           A new TSO command called SHOWTPVT documents all       *   FILE 185
//*           the fields of the TPVT control block (TSO PARMLIB     *   FILE 185
//*           Vector Table) which is not documented publicly        *   FILE 185
//*           by IBM.  SHOWTPVT shows all the values of your        *   FILE 185
//*           own system's TPVT.                                    *   FILE 185
//*                                                                 *   FILE 185
//*           Remember that the member LOADLIB of this pds also     *   FILE 185
//*           contains a load module for a greatly expanded         *   FILE 185
//*           IKJTABLS to authorize programs and commands under     *   FILE 185
//*           TSO.  This load module was created from the ASMTABLS  *   FILE 185
//*           source code.  For TSO/E Release 2.n, and higher,      *   FILE 185
//*           this load module (and its aliases) can be used as     *   FILE 185
//*           is.  There is also plenty of room to zap more names,  *   FILE 185
//*           in the IKJEFTE2, IKJEFTE8, and IKJEFTAP tables.  You  *   FILE 185
//*           may want to zap the tables to authorize more of your  *   FILE 185
//*           favorite programs.  Put it in an APF authorized       *   FILE 185
//*           STEPLIB in your TSO logon proc.  Has to be SETCODE    *   FILE 185
//*           AC(1).                                                *   FILE 185
//*                                                                 *   FILE 185
//*           Updated for z/OS Version 2.3.     (CBT498)            *   FILE 185
//*                                                                 *   FILE 185
~~~~~~~~~~~~~~~~

