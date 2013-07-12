juniper-ex-vcp-to-mib
=====================

        This script will gather several information about VCP links and populate Juniper Mibs entries with them :
        Information gather per VCP link
               - Input/Output Bytes per second  Counter 64
               - CRC Errors                     Counter 64
               - Input/Output Bytes             Counter 64
        
        A VCP link not available for one hour will be automatically removed from Mib 
               
        By default, only vcp-* links will be gather, if you want to get all VCP links including "internal" you must add "ports all" option 
                op ex-vcp-to-mib ports all
               
        To check if Mib is correctly populated, and to get OID for each entry, we can use these commands from CLI : 
                root@EX> show snmp mib walk jnxUtilCounter64Value
                root@EX> show snmp mib walk jnxUtilCounter64Value ascii | match 4743
        
        As a reminder 
                jnxUtilCounter32Value           1.3.6.1.4.1.2636.3.47.1.1.1.1.2
                jnxUtilCounter64Value           1.3.6.1.4.1.2636.3.47.1.1.2.1.2
                jnxUtilIntegerValue             1.3.6.1.4.1.2636.3.47.1.1.3.1.2
                jnxUtilUintValue                1.3.6.1.4.1.2636.3.47.1.1.4.1.2
                jnxUtilStringValue              1.3.6.1.4.1.2636.3.47.1.1.5.1.2

       for-each( $vcport-stat-ext-output/multi-routing-engine-item/virtual-chassis-port-statistics-information/port-list/statistics ) {
                
                
Installation guide
==================         
        This script can run as "OP" or "Events" script
        
        To execute it as "op script"
                Copy it on /var/db/scripts/op on each RE
                Add "set system scripts op file ex-vcp-to-mib.slax"
                
                Then, execute it from cli with 
                        op ex-vcp-to-mib <option>
                        
        To execute it as "Event script"
                 Copy it on /var/db/scripts/event on each RE
                 
                 Declare it into the configuration
                        set event-options policy VCP-TO-MIB events <event-name>
                        set event-options policy VCP-TO-MIB then event-script "ex-vcp-to-mib <option>"
                        
                 if you want to execute it periodically, you can create an event like this:
                        set event-options generate-event 2MIN time-interval 120
                 
                 For the first execution as Event script, it is recommanded to activate debug to syslog
                        set event-options policy VCP-TO-MIB then event-script "ex-vcp-to-mib debug syslog"
                 Then you can monitor log file to confirm if script is running as expected.
                 
Known limitations
=================
        -- on old Junos Version (discover on 10.0), the get-virtual-chassis-port-statistics output is different and the script doesn't get the VCP list. 
        As a workaroud, you have to replace this line
                for-each( $vcport-stat-ext-output/multi-routing-engine-item/virtual-chassis-port-statistics-information/statistics-port-list/statistics ) {
        With this one 
                for-each( $vcport-stat-ext-output/multi-routing-engine-item/virtual-chassis-port-statistics-information/port-list/statistics ) {
        
