

# Session Management Release Notes

## Introduction to Session Management

Session management is a feature of BSP B2C, new to version 3.3. The intent of the new feature is to address security concerns raised by the former method of relaying information back and forth between the client and the server in previous versions of BSP B2C.

### Security issue addressed

Previously, the transfer of information back and forth took place in an environment referred to as “stateless”, which is to say that there was no central location to record the identity of the user or the status of the session. Consequently, there was no way to know when a user had logged out or otherwise left the system, and the session remained technically open until it actually timed out. This made it possible for the user, or anyone else, to simply resume the session (such as by cutting and pasting the URL into a new browser window) anytime before it actually timed out. In addition, since there was no central record of the status of the session, it was possible for the user (or anyone successfully duplicating the user’s session vector) to engage in multiple logins of the system concurrently.

The new, “stateful” session management centrally records, on the server, the user identity and session status when a successful login with appropriate credentials is made. Since the status of the session is centrally known, it becomes possible for the user to perform a true logout of the system, closing the session. The system tracking also means that the number of logins of the system is known, and can be controlled, and even limited.

### Benefits of the new session management

Thanks to the new stateful session management object, a number of new facilities become available, including, but not limited to:

-   knowing exactly who is logged onto the system and how many times concurrently
-   emergency forced logoffs
-   calculating the average session duration (since login and logout times are now know)

    While this new stateful session management system is being applied, initially, to BSP B2C, the intention is to extend this feature to B2B and BSP Small Business as well.

## How It Works

### **Overview** 

As mentioned in the Introduction, session management creates a secure connection between a user and the system by creating an object on the server capable of keeping track of the status of the session once it has been successfully established. To understand the differences between the previous and current stateless and stateful session structures, it is important to understand exactly how each type of session is created and handled.

> [!NOTE]
>
> It is important to point out here that the system is not running two separate applications to address the needs of stateless and stateful session billers, but two modes of the same application: 3.3 stateful mode and 3.2 compatibility mode.*
>

### Architecture of the stateless session

In a stateless session, request handling proceeds as follows:

When a session is initiated, the system checks the user credentials supplied by the user at login. Assuming the criteria are authenticated, the user is authorized and the session is opened.

![](media/Token%20authentication%201.jpg)

Communications between the server and client are affixed with an encrypted “session vector”. This is small token of information, identifying the user and the session, that is attached for purposes of authentication to data moving back and forth between the user and the system. Any data received by the system with this matching session vector is treated as authentic, and the accompanying request is carried out (or at least attempted) and a response is returned to the client.

Since the system contains no centralized means of determining the status of any particular session, the session remains open for whatever pre-determined length of time must elapse, in order for the system to deem the session “timed out”, since the receipt of the last request from the user. In this system, a timing-out of a session is the only way a session can be closed, regardless of the action of the user at the client end

### Architecture of the stateful session

In a stateful session, request handling proceeds as follows:

When a session is initiated, the system checks the user credentials supplied by the user at login. Assuming the criteria are authenticated, the user is authorized and the session is opened.

![](media/Token%20authentication%202.jpg)

The session itself is established as an object on the server, and a key, called a “session handle”, is created and attached to data moving between the server and client. The session handle does not contain information about the user per se, but is an encrypted string which cannot be duplicated, and which must be presented to the server with each request.

Since the session handle is unique, and the system is aware of the status of the session, it is possible to number and limit simultaneous logins by a single user. It also becomes possible for the user to send an end session request, since the system is aware of the state of the session, allowing for a true logout from the system not dependent upon timing out.

## Problems & Troubleshooting

### Overview

Since, for the moment at least, billers using either BSP B2C 3.2 or 3.3 mode will be accessing the server simultaneously, the server must be equipped to intercept and interpret incoming requests and channel them appropriately to either one session management system or the other. This will remain the case until such time as all billers are migrated fully and successfully to the new session management system. Until then, the duality of session handling poses potential problems.

### Misconfiguration

This is a problem most particular to billers who have migrated from the old stateless session management system used in BSP B2C 3.2 and previous versions, to the new stateful session management system used in 3.3. Legacy files and templates in the biller’s installation may be picked up by the system inappropriately and can cause issues with the successful operation of the session flow, or else may cause requests to be misdirected on the server.

### Premature session time-outs

These can occur due either to an invalid configuration of the system during installation, or encoding errors written into the configuration files themselves. Transactions that require an open session are vulnerable to failure by premature session time-outs. Transactions that do not require a session are those such as login and sign-up/self-enrollment.

### Absolute vs. relative URLs

For the purposes of 3.3 stateful mode, HTML pages used by a biller are required to be relative URLs, whereas in 3.2 mode, absolute URLs were permissible. The requirements of the 3.3 stateful mode mean that absolute URLS will cause the system to fail.

> [!NOTE]
>
> Even billers configured to use 3.2 compatability mode must be configured to use a relative URL structure.*
>

## Configuring a Biller

### Notes on biller migration from 3.2  handler to 3.3

Billers intended to be kept with 3.2 handling (3.2 compatable mode) need not to be configured. The default configuration will take care of that.

Extra care should be taken determining what to include into the default client configuration. This will apply to all clients (billers), unless overriden by the specific client configuration.

### For consideration when migrating a biller to 3.3 session management

To convert a biller to 3.3 stateful session management, you need to:

1.  convert HTML pages to relative URLs
2.  convert login procedures to new sign-on procedures (see ABC as the example)
3.  separate HTML pages from templates and put any customized template into etc/services directories
4.  create a client configuration record in the \$SYSROOT/etc/services/ rtctxcfg.xml file
5.  override default session manager by adding

    ```
    <session-manager-ref\>database\</session-manager-ref\> into the client configuration record
    ```

### B2C BSP 3.3 client install notes

Release 3.3 has defined a directory structure and packaging of client installations, and the primary assumption is that a server will have a

\${SYSROOT} defined to be the top level directory, all files and subdirectories are then relative to SYSROOT. Each server is assigned a project (\${PROJECT}): b2c-bc, b2c-cc, b2c-rv-rpt for BillerConsole, CustomerConsole and ReportServer respectively, and all files necessary to correctly operate a server are packaged within the directory structure for that server.

| **B2C BSP 3.3 directory structure** |                                   |                       |             |             |                                                              |
| ----------------------------------- | --------------------------------- | --------------------- | ----------- | ----------- | ------------------------------------------------------------ |
| \${SYSROOT}/etc/                    |                                   |                       |             |             |                                                              |
|                                     | dbcfg.xml                         |                       |             |             | generated during configuration from dbcfg.xml.tmpl           |
|                                     | sv-smcfg.xml                      |                       |             |             | "session vector based session manager, must be default for a system" |
|                                     | db-smcfg.xml                      |                       |             |             | "database based session manager, to be used by 3.3 billers"  |
|                                     | jg-smcfg.xml                      |                       |             |             | JavaGroups distributed session manager                       |
|                                     | services/                         | rtctxcfg.xml          |             |             | runtime context configuration shared by BC and CC            |
|                                     | [b2c-bc\|b2c-cc\|b2c<br>-rv-rpt]/ |                       |             |             | The name of the directory must match the name of the server project |
|                                     |                                   | request-hand ler.xml  |             |             | server process configuration                                 |
|                                     |                                   | logger_config<br>.xml |             |             | server logging setup                                         |
|                                     |                                   | services.xml          |             |             | services hierarchy                                           |
|                                     |                                   | response-ha ndler.xml |             |             | server auditing                                              |
|                                     |                                   | appmenu/              |             |             |                                                              |
|                                     |                                   | error/                |             |             |                                                              |
|                                     |                                   | fmtmpls/              |             |             |                                                              |
|                                     |                                   |                       | [services]/ |             | "1 directory per services with default template, directory names must match directories with service xml description files" |
|                                     |                                   |                       | [clientid]/ |             |                                                              |
|                                     |                                   |                       |             | [services]/ | "1 directory per services, with templates specialized for the client" |
|                                     |                                   |                       | [ib]/       |             |                                                              |
|                                     |                                   |                       |             | default/    | all default legacy templates                                 |
|                                     |                                   |                       |             | [clientid]/ | "all default legacy templates, specialized for a client"     |
|                                     |                                   | ib/                   |             |             |                                                              |
|                                     |                                   | interceptor/          |             |             |                                                              |
|                                     |                                   | presentment/          |             |             |                                                              |
|                                     |                                   | signon/               |             |             |                                                              |
|                                     |                                   | user/                 |             |             |                                                              |
|                                     |                                   | util/                 |             |             |                                                              |

The following is the list of steps required to configure a client for BSP B2C 3.3+:

1.  Follow 3.2 configuration steps
2.  Add a new entry to the end of dbcfg.xml.tmpl \<schema-map-entry client="[clientid]" schema="[tla]"/\>
3.  Add a new client-config entry to rtctxcfg.xml.

    ```
    <client-config applies-to="[clientid]">
        <!-Session Manager Configuration Section->
        <!-Reference to the appropriate session manager -->
        <session-manager-ref>database</session-manager-ref>
        <!-Optional, maximum number of open sessions per user
        <property name="max-sessions-per-user" value="[session limit]"/>
        <!-End of session manager configuration section->
        <!-Seamless login section->
        <!-Required if this client has been previously using 3.2, and is enabled for seamless login->
        <property name="sl-redirect-url" value="true"/>
        <!-Required if the above redirect is enabled, this is the web context for this client->
        <property name="application-context" value="/[tla]/"/>
        <!-Seamless login configuration, one set per channel->
        <parameter-set>
            <!-Possible channels are CSP, SL->
            <parameter-set-name>[channel name]</parameter-set-name>
            <init-from-db>true</init-from-db>
            <property name="sl-token-timeout" value="999999"/>
        </parameter-set>
        <!-End of seamless login section
    </client-config>
    ```
    
4.  *URLs must be relative to the web application context.* Place all specialized 3.2 templates into \${SYSROOT}/etc/services/\${PROJECT}/ fmtmpls/ib/[clientid]
5.  *URLs must be relative to the web application context.* Place all specialized 3.3 (services) templates into \${SYSROOT}/etc/services/

    \${PROJECT}/fmtmpls/[clientid]

6.  Restart application servers.

### Configuring a biller separately

It is possible to create a separate file in which an individual biller can be configured, rather than following the previous procedure (which adds a biller configuration to a general configuration file, namely rtctxcfg.xml. To create a separate biller configuration file, follow these steps:

1.  Create a separate biller configuration file following the naming convention \<billerID\>-cfg.xml (e.g., a biller with the biller ID “701020300” would have a configuration file named “701020300-cfg.xml”).
2.  In the new file, enter and configure the following information:

    ```
    <client-config applies-to="[clientid]">
        <!-Session Manager Configuration Section->
        <!-Reference to the appropriate session manager -->
        <session-manager-ref>database</session-manager-ref>
        <!-Optional, maximum number of open sessions per user à
        <property name="max-sessions-per-user" value="[session limit]"/>
        <!-End of session manager configuration section->
        <!-Seamless login section->
        <!-Required if this client has been previously using 3.2, and is enabled for seamless login->
        <property name="sl-redirect-url" value="true"/>
        <!-Required if the above redirect is enabled, this is the web context for this client->
        <property name="application-context" value="/[tla]/"/>
        <!-Seamless login configuration, one set per channel->
        <parameter-set>
            <!-Possible channels are CSP, SL->
            <parameter-set-name>[channel name]</parameter-set-name>
            <init-from-db>true</init-from-db>
            <property name="sl-token-timeout" value="999999"/>
        </parameter-set>
        <!-End of seamless login section ->
        <!-Additional properties -->
    </client-config>
    
    ```
    
3.  Save the biller configuration file in the \${SYSROOT}/etc/services/ client-config/ directory.

### Configuring rules-based groups hit-counting

New to this release is the specific facility to count the hits made on

rules-based groups in the server. By default, this parameter is deactivated in a typical install, and so is only available to billers particularly configured to use it.

The default parameter resides in the rtctxcfg.xml file, and is set to the value “false”, making it inactive:

```
<property name="calc-target-hits" value="false"/\>
```

To activate this property for *all* billers, the value would be set instead to “true”. Typically, however, this functionality would be activated only for specified billers.

To configure a particular biller to make use of this function, follow these steps:

1.  Create, or open a previously-existing, separate biller configuration file following the naming convention \<billerID\>-cfg.xml (e.g., a biller with the biller ID “701020300” would have a configuration file named “701020300-cfg.xml”).
2. In the biller’s configuration file, enter and configure the following information (after the comment "additional properties" in the file; see example above):

   ```
   <property name="calc-target-hits" value="true"/\>
   ```

3.  Save the biller configuration file in the \${SYSROOT}/etc/services/ client-config/ directory.


### Sample XML configuration file

```
<?xml version="1.0" encoding="utf-8"?>
<runtime-context-config>
    <!-- Database configuration -->
    <db-config include-file="${SYSROOT}/etc/dbcfg.xml"/>
    <!-- Default Session Manager configuration -->
    <sm-config name="default" include-file="${SYSROOT}/etc/ sv-smcfg.xml"/>
    <!-- declaration of additional session managers -->
    <sm-config name="database" include-file="${SYSROOT}/etc/ db-smcfg.xml"/>

    <class-name>com.metavante.to.b2c.services.ib.IBRuntimeContext</ class-name>

    <!-- Seamless login key set root directory -->
    <property name="ksc-root-dir" value="${SYSROOT}/etc/ks/"/>
    <!-- Seamless login configuration file -->
    <property name="ksc-prop-fname" value="config/ksclist.properties"/>

    <!-- declaration of default client configuration -->
    <client-config applies-to="all">
        <session-manager-ref>default</session-manager-ref>
        <property name="dirpath" value="${SYSROOT}/${TMPLROOT}/ib/"/>
        <property name="alt-dirpath" value="${SYSROOT}/etc/services/ b2c-cc/fmtmpls/ib/"/>
        <!-- Customer console path, which is somewhat redundant but required for
        backward compatibility -->
        <property name="customer-path" value="${TMPLROOT}/ib/"/>
        <property name="account-list-type" value="1"/>
        <property name="enable-banner-stats" value="true"/>
        <property name="payment-option" value="0"/>
        <!-- Maximum number of times the same password may be used -->
        <property name="max-use-password" value="-1"/>
        <parameter-set>
            <parameter-set-name>DEFAULT</parameter-set-name>
            <init-from-db>true</init-from-db>
        </parameter-set>
        </client-config>
    <!-- declaration of ABC configuration -->
    <client-config applies-to="701020300">
        <!-- override default session manager for ABC -->
        <session-manager-ref>database</session-manager-ref>
        <!--property name="max-sessions-per-user" value="2"/-->
        <property name="sl-redirect-url" value="true"/>
        <property name="application-context" value="/abc/"/>
        <parameter-set>
            <parameter-set-name>CSP</parameter-set-name>
            <init-from-db>true</init-from-db>
        <property name="sl-token-timeout" value="999999"/>
        </parameter-set>
    </client-config>
</runtime-context-config>
```
