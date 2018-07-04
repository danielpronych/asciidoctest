<p>==============
 SR_Subscribe 
==============</p>
<hr />
<h2 id="select-and-conditionally-download-posted-files">Select and Conditionally Download Posted Files</h2>
<p>:Manual section: 1
:Date: @Date@
:Version: @Version@
:Manual group: Metpx-Sarracenia Suite</p>
<h1 id="synopsis">SYNOPSIS</h1>
<p><strong>sr_subscribe</strong> foreground|start|stop|restart|reload|sanity|status configfile</p>
<p><strong>sr_subscribe</strong> cleanup|declare|setup|disable|enable|list|add|remove configfile</p>
<h1 id="description">DESCRIPTION</h1>
<p>[ <code>version francaise &lt;fr/sr_subscribe.1.md&gt;</code>_ ]</p>
<div class="toc"><span class="toctitle">Contents</span><ul>
<li><a href="#select-and-conditionally-download-posted-files">Select and Conditionally Download Posted Files</a></li>
<li><a href="#synopsis">SYNOPSIS</a></li>
<li><a href="#description">DESCRIPTION</a><ul>
<li><a href="#documentation">Documentation</a></li>
<li><a href="#configurations">Configurations</a></li>
<li><a href="#option-syntax">Option Syntax</a></li>
<li><a href="#log-files">LOG FILES</a></li>
<li><a href="#credentials">CREDENTIALS</a></li>
</ul>
</li>
<li><a href="#consumer">CONSUMER</a><ul>
<li><a href="#setting-the-broker">Setting the Broker</a></li>
<li><a href="#creating-the-queue">Creating the Queue</a></li>
<li><a href="#amqp-queue-bindings">AMQP QUEUE BINDINGS</a></li>
<li><a href="#client-side-filtering">Client-side Filtering</a></li>
<li><a href="#delivery-specifications">DELIVERY SPECIFICATIONS</a></li>
<li><a href="#delivery-completion-inflight">Delivery Completion (inflight)</a></li>
</ul>
</li>
<li><a href="#periodic-processing">PERIODIC PROCESSING</a></li>
<li><a href="#error-recovery">ERROR RECOVERY</a></li>
<li><a href="#examples">EXAMPLES</a></li>
<li><a href="#naming-exchanges-and-queues">NAMING EXCHANGES AND QUEUES</a></li>
<li><a href="#queues-and-multiple-streams">QUEUES and MULTIPLE STREAMS</a></li>
<li><a href="#report_back-and-report_exchange">report_back and report_exchange</a></li>
<li><a href="#logs">LOGS</a></li>
<li><a href="#instances">INSTANCES</a><ul>
<li><a href="#vip-activepassive-options">vip - ACTIVE/PASSIVE OPTIONS</a></li>
</ul>
</li>
<li><a href="#posting-options">POSTING OPTIONS</a><ul>
<li><a href="#-blocksize-default-0-auto">[--blocksize <value>] (default: 0 (auto))</a></li>
<li><a href="#-pbd-post_base_dir-optional">[-pbd|--post_base_dir <path>] (optional)</a></li>
<li><a href="#post_url-mandatory">post_url <url> (MANDATORY)</a></li>
<li><a href="#post_exchange-default-xpublic">post_exchange <name> (default: xpublic)</a></li>
<li><a href="#post_exchange_split-default-0">post_exchange_split   <number>   (default: 0)</a></li>
<li><a href="#remote-configurations">Remote Configurations</a></li>
</ul>
</li>
<li><a href="#pumping">PUMPING</a></li>
<li><a href="#plugin-scripts">PLUGIN SCRIPTS</a></li>
<li><a href="#queue-saverestore">Queue Save/Restore</a><ul>
<li><a href="#sender-destination-unavailable">Sender Destination Unavailable</a></li>
<li><a href="#shovel-saverestore">Shovel Save/Restore</a></li>
</ul>
</li>
<li><a href="#roles-feederadmindeclare">ROLES - feeder/admin/declare</a><ul>
<li><a href="#subscriber">subscriber</a></li>
<li><a href="#source">source</a></li>
</ul>
</li>
<li><a href="#configuration-files">CONFIGURATION FILES</a></li>
<li><a href="#see-also">SEE ALSO</a></li>
<li><a href="#sundew-compatibility-options">SUNDEW COMPATIBILITY OPTIONS</a><ul>
<li><a href="#destfn_script-defaultnone">destfn_script <script> (default:None)</a></li>
<li><a href="#filename-defaultwhatfn">filename <keyword> (default:WHATFN)</a></li>
<li><a href="#field-replacements">Field Replacements</a></li>
<li><a href="#deprecated-settings">DEPRECATED SETTINGS</a></li>
</ul>
</li>
<li><a href="#history">HISTORY</a><ul>
<li><a href="#dd_subscribe-renaming">dd_subscribe Renaming</a></li>
</ul>
</li>
</ul>
</div>
<p>Sr_subscribe is a program to download files from websites or file servers 
that provide <code>sr_post(7) &lt;sr_post.7.md&gt;</code>_ protocol notifications.  Such sites 
publish messages for each file as soon as it is available.  Clients connect to a
<em>broker</em> (often the same as the server itself) and subscribe to the notifications.
The <em>sr_post</em> notifications provide true push notices for web-accessible folders (WAF),
and are far more efficient than either periodic polling of directories, or ATOM/RSS style 
notifications. Sr_subscribe can be configured to post messages after they are downloaded,
to make them available to consumers for further processing or transfers.</p>
<p><strong>sr_subscribe</strong> can also be used for purposes other than downloading, (such as for 
supplying to an external program) specifying the -n (<em>notify_only</em>, or <em>no_download</em>) will
suppress the download behaviour and only post the URL on standard output.  The standard
output can be piped to other processes in classic UNIX text filter style.  </p>
<p>Sr_subscribe is very configurable and is the basis for other components of sarracenia:</p>
<ul>
<li><code>sr_report(1) &lt;sr_report.1.md&gt;</code>_ - process report messages.</li>
<li><code>sr_sender(1) &lt;sr_sender.1.md&gt;</code>_ - copy messages, only, not files.</li>
<li><code>sr_winnow(8) &lt;sr_winnow.8.md&gt;</code>_ - suppress duplicates.</li>
<li><code>sr_shovel(8) &lt;sr_shovel.8.md&gt;</code>_ - copy messages, only, not files.</li>
<li><code>sr_sarra(8) &lt;sr_sarra.8.md&gt;</code>_ -   Subscribe, Acquire, and Recursival ReAdvertise Ad nauseam.</li>
</ul>
<p>All of these components accept the same options, with the same effects.
There is also <code>sr_cpump(1) &lt;sr_cpump.1.md&gt;</code>_ which is a C version that implements a
subset of the options here, but where they are implemented, they have the same effect.</p>
<p>The <strong>sr_subscribe</strong> command takes two arguments: an action start|stop|restart|reload|status, 
followed by an a configuration file. </p>
<p>When any component is invoked, an operation and a configuration file are specified. The operation is one of:</p>
<ul>
<li>foreground: run a single instance in the foreground logging to stderr</li>
<li>restart: stop and then start the configuration.</li>
<li>sanity: looks for instances which have crashed or gotten stuck and restarts them.</li>
<li>start:  start the configuration running</li>
<li>status: check if the configuration is running.</li>
<li>stop: stop the configuration from running</li>
</ul>
<p>Note that the <em>sanity</em> check is invoked by heartbeat processing in sr_audit on a regular basis.
The remaining operations manage the resources (exchanges,queues) used by the component on
the rabbitmq server, or manage the configurations.</p>
<ul>
<li>cleanup:  deletes the component's resources on the server</li>
<li>declare:  creates the component's resources on the server</li>
<li>setup:    like declare, additionally does queue bindings</li>
<li>add:      copy to the list of available configurations.</li>
<li>list:     List all the configurations available.</li>
<li>edit:     modify an existing configuration.</li>
<li>remove:   Remove a configuration</li>
<li>disable:  mark a configuration as ineligible to run. </li>
<li>enable:   mark a configuration as eligible to run. </li>
</ul>
<p>For example:  <em>sr_subscribe foreground dd</em> runs the sr_subscribe component with
the dd configuration as a single foreground instance.</p>
<p>The <strong>foreground</strong> action is used when building a configuration or for debugging.
The <strong>foreground</strong> instance will run regardless of other instances which are currently
running.  Should instances be running, it shares the same message queue with them.
A user stop the <strong>foreground</strong> instance by simply using <ctrl-c> on linux
or use other means to kill the process.</p>
<p>The actions <strong>cleanup</strong>, <strong>declare</strong>, <strong>setup</strong> can be used to manage resources on
the rabbitmq server. The resources are either queues or exchanges. <strong>declare</strong> creates
the resources. <strong>setup</strong> creates and additionally binds the queues.</p>
<p>The <strong>add, remove, list, edit, enable &amp; disable</strong> actions are used to manage the list 
of configurations.  One can see all of the configurations available using the <strong>list</strong>
action.  using the <strong>edit</strong> option, one can work on a particular configuarion.
A <em>disabled</em> configuration will not be started or restarted by the <strong>start</strong>,<br />
<strong>foreground</strong>, or <strong>restart</strong> actions. It can be used to set aside a configuration
temporarily.</p>
<h2 id="documentation">Documentation</h2>
<p>While manual pages provide an index or reference for all options,
new users will find the guides provide more helpful examples and walk 
throughs and should start with them.</p>
<p>users:</p>
<ul>
<li><code>Installation &lt;Install.md&gt;</code>_ - initial installation.</li>
<li><code>Subscriber Guide &lt;subscriber.md&gt;</code>_ - effective downloading from a pump.</li>
<li><code>Source Guide &lt;source.md&gt;</code>_ - effective uploading to a pump</li>
<li><code>Programming Guide &lt;Prog.md&gt;</code>_ - Programming custom plugins for workflow integration.</li>
</ul>
<p>Administrators:</p>
<ul>
<li><code>Admin Guide &lt;Admin.md&gt;</code>_ - Configuration of Pumps</li>
<li><code>Upgrade Guide &lt;UPGRADING.md&gt;</code>_ - MUST READ when upgrading pumps.</li>
</ul>
<p>contributors:</p>
<ul>
<li><code>Developer Guide &lt;Dev.md&gt;</code>_ - contributing to sarracenia development.</li>
</ul>
<p>meta:</p>
<ul>
<li><code>Overview &lt;sarra.md&gt;</code>_ - Introduction.</li>
<li><code>Concepts &lt;Concepts.md&gt;</code>_ - Concepts and Glossary</li>
</ul>
<p>There are also other manual pages available here: <code>See Also</code>_</p>
<p>Some quick hints are also available When the command line is invoked with 
either the <em>help</em> action, or <em>-help</em> op <strong>help</strong> to have a component print 
a list of valid options. </p>
<h2 id="configurations">Configurations</h2>
<p>If one has a ready made configuration called <em>q_f71.conf</em>, it can be 
added to the list of known ones with::</p>
<p>sr_subscribe add q_f71.conf</p>
<p>In this case, xvan_f14 is included with examples provided, so <em>add</em> finds it in the examples
directory and copies into the active configuration one. 
Each configuration file manages the consumers for a single queue on
the broker. To view the available configurations, use::</p>
<p>blacklab% sr_subscribe list</p>
<p>packaged plugins: ( /usr/lib/python3/dist-packages/sarra/plugins ) 
         <strong>pycache</strong>       bad_plugin1.py       bad_plugin2.py       bad_plugin3.py     destfn_sample.py       download_cp.py 
      download_dd.py      download_scp.py     download_wget.py          file_age.py        file_check.py          file_log.py 
      file_rxpipe.py        file_total.py           harness.py          hb_cache.py            hb_log.py         hb_memory.py 
         hb_pulse.py         html_page.py          line_log.py         line_mode.py               log.py         msg_2http.py 
       msg_2local.py    msg_2localfile.py     msg_auditflow.py     msg_by_source.py       msg_by_user.py         msg_delay.py 
       msg_delete.py      msg_download.py          msg_dump.py        msg_fdelay.py msg_filter_wmo2msc.py  msg_from_cluster.py 
    msg_hour_tree.py           msg_log.py     msg_print_lag.py   msg_rename4jicc.py    msg_rename_dmf.py msg_rename_whatfn.py 
      msg_renamer.py msg_replace_new_dir.py          msg_save.py      msg_skip_old.py        msg_speedo.py msg_sundew_pxroute.py 
   msg_test_retry.py   msg_to_clusters.py         msg_total.py        part_check.py  part_clamav_scan.py        poll_pulse.py 
      poll_script.py    post_hour_tree.py          post_log.py    post_long_flow.py     post_override.py   post_rate_limit.py 
       post_total.py         watch_log.py </p>
<p>configuration examples: ( /usr/lib/python3/dist-packages/sarra/examples/subscribe ) 
            all.conf     all_but_cap.conf            amis.conf            aqhi.conf             cap.conf      cclean_f91.conf 
      cdnld_f21.conf       cfile_f44.conf        citypage.conf       clean_f90.conf            cmml.conf cscn22_bulletins.conf 
        ftp_f70.conf            gdps.conf         ninjo-a.conf           q_f71.conf           radar.conf            rdps.conf 
           swob.conf           t_f30.conf      u_sftp_f60.conf </p>
<p>user plugins: ( /home/peter/.config/sarra/plugins ) 
        destfn_am.py         destfn_nz.py       msg_tarpush.py </p>
<p>general: ( /home/peter/.config/sarra ) 
          admin.conf     credentials.conf         default.conf</p>
<p>user configurations: ( /home/peter/.config/sarra/subscribe )
     cclean_f91.conf       cdnld_f21.conf       cfile_f44.conf       clean_f90.conf         ftp_f70.conf           q_f71.conf 
          t_f30.conf      u_sftp_f60.conf
  blacklab%</p>
<p>one can then modify it using::</p>
<p>sr_subscribe edit q_f71.conf</p>
<p>(The edit command uses the EDITOR environment variable, if present.)
Once satisfied, one can start the the configuration running::</p>
<p>sr_subscibe foreground q_f71.conf</p>
<p>What goes into the files? See next section:</p>
<h2 id="option-syntax">Option Syntax</h2>
<p>Options are placed in configuration files, one per line, in the form:</p>
<p><strong>option <value></strong></p>
<p>For example::</p>
<p><strong>debug true</strong>
  <strong>debug</strong></p>
<p>sets the <em>debug</em> option to enable more verbose logging.  If no value is specified,
the value true is implicit. so the above are equivalent.  An second example 
configuration line::</p>
<p>broker amqp://anonymous@dd.weather.gc.ca</p>
<p>In the above example, <em>broker</em> is the option keyword, and the rest of the line is the 
value assigned to the setting. Configuration files are a sequence of settings, one per line. 
Note that the files are read in order, most importantly for <em>directory</em> and <em>accept</em> clauses.<br />
Example::</p>
<pre><code>directory A
accept X
</code></pre>
<p>Places files matching X in directory A.</p>
<p>vs::
    accept X
    directory A</p>
<p>Places files matching X in the current working directory, and the <em>directory A</em> setting 
does nothing in relation to X.</p>
<p>To provide non-functional description of configuration, or comments, use lines that begin with a <strong>#</strong>.</p>
<p><strong>All options are case sensitive.</strong>  <strong>Debug</strong> is not the same as <strong>debug</strong> or <strong>DEBUG</strong>.
Those are three different options (two of which do not exist and will have no effect,
but should generate an ��unknown option warning��.)</p>
<p>Options and command line arguments are equivalent.  Every command line argument
has a corresponding long version starting with '--'.  For example <em>-u</em> has the
long form <em>--url</em>. One can also specify this option in a configuration file.
To do so, use the long form without the '--', and put its value separated by a space.
The following are all equivalent:</p>
<ul>
<li><strong>url <url></strong></li>
<li><strong>-u <url></strong></li>
<li><strong>--url <url></strong></li>
</ul>
<p>Settings in an individual .conf file are read in after the default.conf
file, and so can override defaults.   Options specified on
the command line override configuration files.</p>
<p>Settings are interpreted in order.  Each file is read from top to bottom.
for example:</p>
<p>sequence #1::</p>
<p>reject .<em>.gif
  accept .</em></p>
<p>sequence #2::</p>
<p>accept .<em>
  reject .</em>.gif</p>
<p>.. note::
   FIXME: does this match only files ending in 'gif' or should we add a $ to it?
   will it match something like .gif2 ? is there an assumed .* at the end?</p>
<p>In sequence #1, all files ending in 'gif' are rejected. In sequence #2, the 
accept .* (which accepts everything) is encountered before the reject statement, 
so the reject has no effect.</p>
<p>Several options that need to be reused in different config file can be grouped in a file.
In each config where the options subset should appear, the user would then use :</p>
<ul>
<li><strong>--include <includeConfigPath></strong></li>
</ul>
<p>The includeConfigPath would normally reside under the same config dir of its
master configs. There is no restriction, any option can be placed in a config file
included. The user must be aware that, for many options, several declarations
means overwriting their values.</p>
<p>plugin options
~~~~~~~~~~~~~~</p>
<p>Sarracenia makes extensive use of small python code snippets that customize
processing called <em>plugins.</em> Plugins define and use additional settings,
that are usually prefixed with the name of the plugin::</p>
<p>msg_to_clusters DDI
  msg_to_clusters DD</p>
<p>on_message msg_to_clusters</p>
<p>A setting 'msg_to_clusters' is needed by the <em>msg_to_clusters</em> plugin
referenced in the <em>on_message</em></p>
<p>Environment Variables
~~~~~~~~~~~~~~~~~~~~~</p>
<p>On can also reference environment variables in configuration files,
using the <em>${ENV}</em> syntax.  If Sarracenia routines needs to make use
of an environment variable, then they can be set in configuration files::</p>
<p>declare env HTTP_PROXY=localhost</p>
<h2 id="log-files">LOG FILES</h2>
<p>As sr_subscribe usually runs as a daemon (unless invoked in <em>foreground</em> mode) 
one normally examines its log file to find out how processing is going.  When only
a single instance is running, one can normally view the log of the running process
like so::</p>
<p>sr_subscribe log <em>myconfig</em></p>
<p>Where <em>myconfig</em> is the name of the running configuration. Log files 
are placed as per the XDG Open Directory Specification. There will be a log file 
for each <em>instance</em> (download process) of an sr_subscribe process running the myflow configuration::</p>
<p>in linux: ~/.cache/sarra/log/sr_subscribe_myflow_01.log</p>
<p>One can override placement on linux by setting the XDG_CACHE_HOME environment variable.</p>
<h2 id="credentials">CREDENTIALS</h2>
<p>One normally does not specify passwords in configuration files.  Rather they are placed 
in the credentials file::</p>
<p>sr_subscribe edit credentials</p>
<p>For every url specified that requires a password, one places 
a matching entry in credentials.conf.
The broker option sets all the credential information to connect to the  <strong>RabbitMQ</strong> server </p>
<ul>
<li><strong>broker amqp{s}://<user>:<pw>@<brokerhost>[:port]/<vhost></strong></li>
</ul>
<p>::</p>
<pre><code>  (default: amqp://anonymous:anonymous@dd.weather.gc.ca/ )
</code></pre>
<p>For all <strong>sarracenia</strong> programs, the confidential parts of credentials are stored
only in ~/.config/sarra/credentials.conf.  This includes the destination and the broker
passwords and settings needed by components.  The format is one entry per line.  Examples:</p>
<ul>
<li><strong>amqp://user1:password1@host/</strong></li>
<li>
<p><strong>amqps://user2:password2@host:5671/dev</strong></p>
</li>
<li>
<p><strong>sftp://user5:password5@host</strong></p>
</li>
<li>
<p><strong>sftp://user6:password6@host:22  ssh_keyfile=/users/local/.ssh/id_dsa</strong></p>
</li>
<li>
<p><strong>ftp://user7:password7@host  passive,binary</strong></p>
</li>
<li>
<p><strong>ftp://user8:password8@host:2121  active,ascii</strong></p>
</li>
<li>
<p><strong>ftps://user7:De%3Aize@host  passive,binary,tls</strong></p>
</li>
<li><strong>ftps://user8:%2fdot8@host:2121  active,ascii,tls,prot_p</strong></li>
</ul>
<p>In other configuration files or on the command line, the url simply lacks the
password or key specification.  The url given in the other files is looked
up in credentials.conf.</p>
<p>Note::
 SFTP credentials are optional, in that sarracenia will look in the .ssh directory
 and use the normal SSH credentials found there.</p>
<p>These strings are URL encoded, so if an account has a password with a special 
 character, its URL encoded equivalent can be supplied.  In the last example above, 
 <strong>%2f</strong> means that the actual password isi: <strong>/dot8</strong>
 The next to last password is:  <strong>De:olonize</strong>. ( %3a being the url encoded value for a colon character. )</p>
<h1 id="consumer">CONSUMER</h1>
<p>Most Metpx Sarracenia components loop on reception and consumption of sarracenia 
AMQP messages.  Usually, the messages of interest are <code>sr_post(7) &lt;sr_post.7.md&gt;</code><em> 
messages, announcing the availability of a file by publishing it��s URL ( or a part 
of a file ), but there are also <code>sr_report(7) &lt;sr_report.7.md&gt;</code></em> messages which 
can be processed using the same tools. AMQP messages are published to an exchange 
on a broker (AMQP server.) The exchange delivers messages to queues. To receive 
messages, one must provide the credentials to connect to the broker (AMQP message 
pump). Once connected, a consumer needs to create a queue to hold pending messages.
The consumer must then bind the queue to one or more exchanges so that they put 
messages in its queue.</p>
<p>Once the bindings are set, the program can receive messages. When a message is received,
further filtering is possible using regular expression onto the AMQP messages.
After a message passes this selection process, and other internal validation, the
component can run an <strong>on_message</strong> plugin script to perform additional message 
processing. If this plugin returns False, the message is discarded. If True, 
processing continues.</p>
<p>The following sections explains all the options to set this "consuming" part of
sarracenia programs.</p>
<h2 id="setting-the-broker">Setting the Broker</h2>
<p><strong>broker amqp{s}://<user>:<password>@<brokerhost>[:port]/<vhost></strong></p>
<p>An AMQP URI is used to configure a connection to a message pump (aka AMQP broker.)
Some sarracenia components set a reasonable default for that option. 
You provide the normal user,host,port of connections. In most configuration files,
the password is missing. The password is normally only included in the credentials.conf file.</p>
<p>Sarracenia work has not used vhosts, so <strong>vhost</strong> should almost always be <strong>/</strong>.</p>
<p>for more info on the AMQP URI format: ( https://www.rabbitmq.com/uri-spec.html )</p>
<p>either in the default.conf or each specific configuration file.
The broker option tell each component which broker to contact.</p>
<p><strong>broker amqp{s}://<user>:<pw>@<brokerhost>[:port]/<vhost></strong></p>
<p>::
      (default: None and it is mandatory to set it ) </p>
<p>Once connected to an AMQP broker, the user needs to bind a queue
to exchanges and topics to determine the messages of interest.</p>
<h2 id="creating-the-queue">Creating the Queue</h2>
<p>Once connected to an AMQP broker, the user needs to create a queue.</p>
<p>Setting the queue on broker :</p>
<ul>
<li><strong>queue_name    <name>         (default: q_<brokerUser>.<programName>.<configName>)</strong></li>
<li><strong>durable       <boolean>      (default: False)</strong></li>
<li><strong>expire        <duration>      (default: 5m  == five minutes. RECOMMEND OVERRIDING)</strong></li>
<li><strong>message-ttl   <duration>      (default: None)</strong></li>
<li><strong>prefetch      <N>            (default: 1)</strong></li>
<li><strong>reset         <boolean>      (default: False)</strong></li>
<li><strong>restore       <boolean>      (default: False)</strong></li>
<li><strong>restore_to_queue <queuename> (default: None)</strong></li>
<li><strong>save          <boolean>      (default: False)</strong></li>
</ul>
<p>Usually components guess reasonable defaults for all these values
and users do not need to set them.  For less usual cases, the user
may need to override the defaults.  The queue is where the notifications
are held on the server for each subscriber.</p>
<p>[ queue_name|qn <name>]
~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>By default, components create a queue name that should be unique. The default queue_name
components create follows :  <strong>q_<brokerUser>.<programName>.<configName></strong> .
Users can override the defaul provided that it starts with <strong>q_<brokerUser></strong>.
Some variables can also be used within the queue_name like
<strong>${BROKER_USER},${PROGRAM},${CONFIG},${HOSTNAME}</strong></p>
<p>durable <boolean> (default: False)
-~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>durable</strong> option, if set to True, means writes the queue
on disk if the broker is restarted.</p>
<p>expire <duration> (default: 5m  == five minutes. RECOMMEND OVERRIDING)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>expire</strong>  option is expressed as a duration... it sets how long should live
a queue without connections. A raw integer is expressed in seconds, if the suffix m,h.d,w
are used, then the interval is in minutes, hours, days, or weeks. After the queue expires,
the contents is dropped, and so gaps in the download data flow can arise.  A value of
1d (day) or 1w (week) can be appropriate to avoid data loss. It depends on how long
the subscriber is expected to shutdown, and not suffer data loss.</p>
<p>The <strong>expire</strong> setting must be overridden for operational use. 
The default is set low because it defines how long resources on the broker will be assigned,
and in early use (when default was 1 week) brokers would often get overloaded with very 
long queues for left-over experiments.  </p>
<p>message-ttl <duration>  (default: None)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>message-ttl</strong>  option set the time a message can live in the queue.
Past that time, the message is taken out of the queue by the broker.</p>
<p>prefetch <N> (default: 1)
~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>prefetch</strong> option sets the number of messages to fetch at one time.
When multiple instances are running and prefetch is 4, each instance will obtain upto four
messages at a time.  To minimize the number of messages lost if an instance dies and have
optimal load sharing, the prefetch should be set as low as possible.  However, over long
haul links, it is necessary to raise this number, to hide round-trip latency, so a setting
of 10 or more may be needed.</p>
<p>reset <boolean> (default: False)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>When <strong>reset</strong> is set, and a component is (re)started, its queue is
deleted (if it already exists) and recreated according to the component's
queue options.  This is when a broker option is modified, as the broker will
refuse access to a queue declared with options that differ from what was
set at creation.  It can also be used to discard a queue quickly when a receiver 
has been shut down for a long period. if duplicate suppression is active, then
the reception cache is also discarded.</p>
<p>The AMQP protocol defines other queue options which are not exposed
via sarracenia, because sarracenia itself picks appropriate values.</p>
<p>save/restore
~~~~~~~~~~~~</p>
<p>The <strong>save</strong> option is used to read messages from the queue and write them
to a local file, saving them for future processing, rather than processing
them immediately.  See the <code>Sender Destination Unavailable</code>_ section for more details.
The <strong>restore</strong> option implements the reverse function, reading from the file
for processing.  </p>
<p>If <strong>restore_to_queue</strong> is specified, then rather than triggering local
processing, the messages restored are posted to a temporary exchange 
bound to the given queue.  For an example, see <code>Shovel Save/Restore</code>_ </p>
<h2 id="amqp-queue-bindings">AMQP QUEUE BINDINGS</h2>
<p>Once one has a queue, it must be bound to an exchange.
Users almost always need to set these options. Once a queue exists
on the broker, it must be bound to an exchange. Bindings define which
messages (URL notifications) the program receives. The root of the topic
tree is fixed to indicate the protocol version and type of the
message (but developers can override it with the <strong>topic_prefix</strong>
option.)</p>
<p>These options define which messages (URL notifications) the program receives:</p>
<ul>
<li><strong>exchange      <name>         (default: xpublic)</strong> </li>
<li><strong>exchange_suffix      <name>  (default: None)</strong> </li>
<li><strong>topic_prefix  <amqp pattern> (default: v02.post -- developer option)</strong> </li>
<li><strong>subtopic      <amqp pattern> (subtopic need to be set)</strong> </li>
</ul>
<p>exchange <name> (default: xpublic) and exchange_suffix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The convention on data pumps is to use the <em>xpublic</em> exchange. Users can establish
private data flow for their own processing. Users can declare their own exchanges
that always begin with <em>xs_<username></em>, so to save having to specify that each
time, one can just set <em>exchange_suffix kk</em> which will result in the exchange
being set to <em>xs_<username>_kk</em> (overriding the <em>xpublic</em> default.) </p>
<p>subtopic <amqp pattern> (subtopic need to be set)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>Several topic options may be declared. To give a correct value to the subtopic,
one has the choice of filtering using <strong>subtopic</strong> with only AMQP's limited wildcarding and
length limited to 255 encoded bytes, or the more powerful regular expression 
based  <strong>accept/reject</strong>  mechanisms described below. The difference being that the 
AMQP filtering is applied by the broker itself, saving the notices from being delivered 
to the client at all. The  <strong>accept/reject</strong>  patterns apply to messages sent by the 
broker to the subscriber. In other words,  <strong>accept/reject</strong>  are client side filters, 
whereas <strong>subtopic</strong> is server side filtering.  </p>
<p>It is best practice to use server side filtering to reduce the number of announcements sent
to the client to a small superset of what is relevant, and perform only a fine-tuning with the 
client side mechanisms, saving bandwidth and processing for all.</p>
<p>topic_prefix is primarily of interest during protocol version transitions, 
where one wishes to specify a non-default protocol version of messages to 
subscribe to. </p>
<p>Usually, the user specifies one exchange, and several subtopic options.
<strong>Subtopic</strong> is what is normally used to indicate messages of interest.
To use the subtopic to filter the products, match the subtopic string with
the relative path of the product.</p>
<p>For example, consuming from DD, to give a correct value to subtopic, one can
browse the our website  <strong>http://dd.weather.gc.ca</strong> and write down all directories
of interest.  For each directory tree of interest, write a  <strong>subtopic</strong>
option as follow:</p>
<p><strong>subtopic  directory1.<em>.subdirectory3.</em>.subdirectory5.#</strong></p>
<p>::</p>
<p>where:<br />
       *                matches a single directory name 
       #                matches any remaining tree of directories.</p>
<p>note:
  When directories have these wild-cards, or spaces in their names, they 
  will be URL-encoded ( '#' becomes %23 )
  When directories have periods in their name, this will change
  the topic hierarchy.</p>
<p>FIXME: 
      hash marks are URL substituted, but did not see code for other values.
      Review whether asterisks in directory names in topics should be URL-encoded.
      Review whether periods in directory names in topics should be URL-encoded.</p>
<h2 id="client-side-filtering">Client-side Filtering</h2>
<p>We have selected our messages through <strong>exchange</strong>, <strong>subtopic</strong> and
perhaps patterned  <strong>subtopic</strong> with AMQP's limited wildcarding which
is all done by the broker (server-side.) The broker puts the 
corresponding messages in our queue. The subscribed component 
downloads the these messages.  Once the message is downloaded, Sarracenia 
clients apply more flexible client side filtering using regular expressions.</p>
<p>Brief Introduction to Regular Expressions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>Regular expressions are a very powerful way of expressing pattern matches. 
They provide extreme flexibility, but in these examples we will only use a
very trivial subset: The . is a wildcard matching any single character. If it
is followed by an occurrence count, it indicates how many letters will match
the pattern. the * (asterisk) character, means any number of occurrences.
so:</p>
<ul>
<li>.* means any sequence of characters of any length. In other words, match anything.</li>
<li>cap.* means any sequence of characters that starts with cap.</li>
<li>.<em>CAP.</em> means any sequence of characters with CAP somewhere in it. </li>
<li>.*cap means any sequence of characters that ends with CAP.  In case where multiple portions of the string could match, the longest one is selected.</li>
<li>.<em>?cap same as above, but </em>non-greedy*, meaning the shortest match is chosen.</li>
</ul>
<p>Please consult various internet resources for more information on the full
variety of matching possible with regular expressions:</p>
<ul>
<li>https://docs.python.org/3/library/re.html</li>
<li>https://en.wikipedia.org/wiki/Regular_expression</li>
<li>http://www.regular-expressions.info/ </li>
</ul>
<p>accept, reject and accept_unmatch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<ul>
<li><strong>accept    <regexp pattern> (optional)</strong></li>
<li><strong>reject    <regexp pattern> (optional)</strong></li>
<li><strong>accept_unmatch   <boolean> (default: False)</strong></li>
</ul>
<p>The  <strong>accept</strong>  and  <strong>reject</strong>  options process regular expressions (regexp).
The regexp is applied to the the message's URL for a match.</p>
<p>If the message's URL of a file matches a <strong>reject</strong>  pattern, the message
is acknowledged as consumed to the broker and skipped.</p>
<p>One that matches an <strong>accept</strong> pattern is processed by the component.</p>
<p>In many configurations, <strong>accept</strong> and <strong>reject</strong> options are mixed
with the <strong>directory</strong> option.  They then relate accepted messages
to the <strong>directory</strong> value they are specified under.</p>
<p>After all <strong>accept</strong> / <strong>reject</strong>  options are processed, normally
the message acknowledged as consumed and skipped. To override that
default, set <strong>accept_unmatch</strong> to True. The <strong>accept/reject</strong> 
settings are interpreted in order. Each option is processed orderly 
from top to bottom. For example:</p>
<p>sequence #1::</p>
<p>reject .<em>.gif
  accept .</em></p>
<p>sequence #2::</p>
<p>accept .<em>
  reject .</em>.gif</p>
<p>In sequence #1, all files ending in 'gif' are rejected.  In sequence #2, the accept .* (which
accepts everything) is encountered before the reject statement, so the reject has no effect.</p>
<p>It is best practice to use server side filtering to reduce the number of announcements sent
to the component to a small superset of what is relevant, and perform only a fine-tuning with the
client side mechanisms, saving bandwidth and processing for all. more details on how
to apply the directives follow:</p>
<h2 id="delivery-specifications">DELIVERY SPECIFICATIONS</h2>
<p>These options set what files the user wants and where it will be placed,
and under which name.</p>
<ul>
<li><strong>accept    <regexp pattern> (must be set)</strong> </li>
<li><strong>accept_unmatch   <boolean> (default: off)</strong></li>
<li><strong>attempts     <count>          (default: 3)</strong></li>
<li><strong>batch     <count>          (default: 100)</strong></li>
<li><strong>default_mode     <octalint>       (default: 0 - umask)</strong></li>
<li><strong>default_dir_mode <octalint>       (default: 0755)</strong></li>
<li><strong>delete    <boolean>&gt;       (default: off)</strong></li>
<li><strong>directory <path>           (default: .)</strong> </li>
<li><strong>discard   <boolean>        (default: off)</strong></li>
<li><strong>base_dir <path>       (default: /)</strong></li>
<li><strong>flatten   <string>         (default: '/')</strong> </li>
<li><strong>heartbeat <count>                 (default: 300 seconds)</strong></li>
<li><strong>inplace       <boolean>        (default: On)</strong></li>
<li><strong>kbytes_ps <count>               (default: 0)</strong></li>
<li><strong>inflight  <string>         (default: .tmp or NONE if post_broker set)</strong> </li>
<li><strong>mirror    <boolean>        (default: off)</strong> </li>
<li><strong>no_download|notify_only    <boolean>        (default: off)</strong> </li>
<li><strong>outlet    post|json|url    (default: post)</strong> </li>
<li><strong>overwrite <boolean>        (default: off)</strong> </li>
<li><strong>recompute_chksum <boolean> (default: off)</strong></li>
<li><strong>reject    <regexp pattern> (optional)</strong> </li>
<li><strong>retry    <boolean>         (default: On)</strong> </li>
<li><strong>retry_ttl    <duration>         (default: same as expire)</strong> </li>
<li><strong>source_from_exchange  <boolean> (default: off)</strong></li>
<li><strong>strip     <count|regexp>   (default: 0)</strong></li>
<li><strong>suppress_duplicates   <off|on|999>     (default: off)</strong></li>
<li><strong>timeout     <float>         (default: 0)</strong></li>
</ul>
<p>attempts <count> (default: 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>attempts</strong> option indicates how many times to 
attempt downloading the data before giving up.  The default of 3 should be appropriate 
in most cases.  When the <strong>retry</strong> option is false, the file is then dropped immediately.</p>
<p>When The <strong>retry</strong> option is set (default), a failure to download after prescribed number
of <strong>attempts</strong> (or send, in a sender) will cause the message to be added to a queue file 
for later retry.  When there are no messages ready to consume from the AMQP queue, 
the retry queue will be queried.</p>
<p>retry_ttl <duration> (default: same as expire)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>retry_ttl</strong> (retry time to live) option indicates how long to keep trying to send 
a file before it is aged out of a the queue.  Default is two days.  If a file has not 
been transferred after two days of attempts, it is discarded.</p>
<p>timeout <float> (default: 0)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>timeout</strong> option, sets the number of seconds to wait before aborting a
connection or download transfer (applied per buffer during transfer.)</p>
<p>inflight <string> (default: .tmp or NONE if post_broker set)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>inflight</strong>  option sets how to ignore files when they are being transferred
or (in mid-flight betweeen two systems.) Incorrect setting of this option causes
unreliable transfers, and care must be taken.  See <code>Delivery Completion (inflight)</code>_ for more details.</p>
<p>The value can be a file name suffix, which is appended to create a temporary name during 
the transfer.  If <strong>inflight</strong>  is set to <strong>.</strong>, then it is a prefix, to conform with 
the standard for "hidden" files on unix/linux.<br />
If <strong>inflight</strong>  ends in / (example: <em>tmp/</em> ), then it is a prefix, and specifies a 
sub-directory of the destination into which the file should be written while in flight. </p>
<p>Whether a prefix or suffix is specified, when the transfer is 
complete, the file is renamed to it's permanent name to allow further processing.</p>
<p>The  <strong>inflight</strong>  option can also be specified as a time interval, for example, 
10 for 10 seconds.  When set to a time interval, a reader of a file ensures that 
it waits until the file has not been modified in that interval. So a file will 
not be processed until it has stayed the same for at least 10 seconds. </p>
<p>Lastly, <strong>inflight</strong> can be set to <em>NONE</em>, which case the file is written directly
with the final name, where the recipient will wait to receive a post notifying it
of the file's arrival.  This is the fastest, lowest overhead option when it is available.
It is also the default when a <em>post_broker</em> is given, indicating that some
other process is to be notified after delivery.</p>
<p>delete <boolean> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>When the <strong>delete</strong> option is set, after a download has completed successfully, the subscriber
will delete the file at the upstream source.  Default is false.</p>
<p>batch <count> (default: 100)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>batch</strong> option is used to indicate how many files should be transferred 
over a connection, before it is torn down, and re-established.  On very low 
volume transfers, where timeouts can occur between transfers, this should be
lowered to 1.  For most usual situations the default is fine. for higher volume
cases, one could raise it to reduce transfer overhead. It is only used for file
transfer protocols, not HTTP ones at the moment.</p>
<p>directory <path> (default: .)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <em>directory</em> option defines where to put the files on your server.
Combined with  <strong>accept</strong> / <strong>reject</strong>  options, the user can select the
files of interest and their directories of residence. (see the  <strong>mirror</strong>
option for more directory settings).</p>
<p>The  <strong>accept</strong>  and  <strong>reject</strong>  options use regular expressions (regexp) to match URL.
Theses options are processed sequentially. 
The URL of a file that matches a  <strong>reject</strong>  pattern is never downloaded.
One that match an  <strong>accept</strong>  pattern is downloaded into the directory
declared by the closest  <strong>directory</strong>  option above the matching  <strong>accept</strong> option.
<strong>accept_unmatch</strong> is used to decide what to do when no reject or accept clauses matched.</p>
<p>::</p>
<p>ex.   directory /mylocaldirectory/myradars
        accept    .<em>RADAR.</em></p>
<pre><code>    directory /mylocaldirectory/mygribs
    reject    .*Reg.*
    accept    .*GRIB.*
</code></pre>
<p>mirror <boolean> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>mirror</strong>  option can be used to mirror the dd.weather.gc.ca tree of the files.
If set to  <strong>True</strong>  the directory given by the  <strong>directory</strong>  option
will be the basename of a tree. Accepted files under that directory will be
placed under the subdirectory tree leaf where it resides under dd.weather.gc.ca.
For example retrieving the following url, with options::</p>
<p>http://dd.weather.gc.ca/radar/PRECIP/GIF/WGJ/201312141900_WGJ_PRECIP_SNOW.gif</p>
<p>mirror    True
   directory /mylocaldirectory
   accept    .<em>RADAR.</em></p>
<p>would result in the creation of the directories and the file
/mylocaldirectory/radar/PRECIP/GIF/WGJ/201312141900_WGJ_PRECIP_SNOW.gif</p>
<p>strip <count|regexp> (default: 0)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>You can modify the mirrored directoties with the <strong>strip</strong> option. 
If set to N  (an integer) the first 'N' directories are removed.
For example ::</p>
<p>http://dd.weather.gc.ca/radar/PRECIP/GIF/WGJ/201312141900_WGJ_PRECIP_SNOW.gif</p>
<p>mirror    True
   strip     3
   directory /mylocaldirectory
   accept    .<em>RADAR.</em></p>
<p>would result in the creation of the directories and the file
/mylocaldirectory/WGJ/201312141900_WGJ_PRECIP_SNOW.gif
when a regexp is provide in place of a number, it indicates a pattern to be removed
from the relative path.  for example if::</p>
<p>strip  .*?GIF/</p>
<p>Will also result in the file being placed the same location. </p>
<p>NOTE::
    with <strong>strip</strong>, use of <strong>?</strong> modifier (to prevent regular expression <em>greediness</em> ) is often helpful. 
    It ensures the shortest match is used.</p>
<pre><code>For example, given a file name:  radar/PRECIP/GIF/WGJ/201312141900_WGJ_PRECIP_SNOW.GIF
The expression:  .*?GIF   matches: radar/PRECIP/GIF
whereas the expression: .*GIF matches the entire name.
</code></pre>
<p>flatten <string> (default: '/')
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>flatten</strong>  option is use to set a separator character. The default value ( '/' )
nullifies the effect of this option.  This character replaces the '/' in the url 
directory and create a "flatten" filename form its dd.weather.gc.ca path.<br />
For example retrieving the following url, with options::</p>
<p>http://dd.weather.gc.ca/model_gem_global/25km/grib2/lat_lon/12/015/CMC_glb_TMP_TGL_2_latlon.24x.24_2013121612_P015.grib2</p>
<p>flatten   -
   directory /mylocaldirectory
   accept    .<em>model_gem_global.</em></p>
<p>would result in the creation of the filepath::</p>
<p>/mylocaldirectory/model_gem_global-25km-grib2-lat_lon-12-015-CMC_glb_TMP_TGL_2_latlon.24x.24_2013121612_P015.grib2</p>
<p>One can also specify variable substitutions to be performed on arguments to the directory 
option, with the use of <em>${..}</em> notation::</p>
<p>SOURCE   - the amqp user that injected data (taken from the message.)
   BD       - the base directory
   PBD      - the post base dir
   YYYYMMDD - the current daily timestamp.
   HH       - the current hourly timestamp.
   <em>var</em>    - any environment variable.</p>
<p>The YYYYMMDD and HH time stamps refer to the time at which the data is processed by
the component, it is not decoded or derived from the content of the files delivered.
All date/times in Sarracenia are in UTC.</p>
<p>Refer to <em>source_from_exchange</em> for a common example of usage.  Note that any sarracenia
built-in value takes precedence over a variable of the same name in the environment.</p>
<p>base_dir <path> (default: /)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p><strong>base_dir</strong> supplies the directory path that, when combined with the relative
one in the selected notification gives the absolute path of the file to be sent.
The defaults is None which means that the path in the notification is the absolute one.</p>
<p><strong>FIXME</strong>::
    cannot explain this... do not know what it is myself. This is taken from sender.
    in a subscriber, if it is set... will it download? or will it assume it is local?
    in a sender.</p>
<p>inplace <boolean> (default: On)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>Large files may be sent as a series of parts, rather than all at once.
When downloading, if <strong>inplace</strong> is true, these parts will be appended to the file 
in an orderly fashion. Each part, after it is inserted in the file, is announced to subscribers.
This can be set to false for some deployments of sarracenia where one pump will 
only ever see a few parts, and not the entirety, of multi-part files. </p>
<p>The <strong>inplace</strong> option defaults to True. 
Depending of <strong>inplace</strong> and if the message was a part, the path can
change again (adding a part suffix if necessary).</p>
<p>outlet post|json|url (default: post)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>outlet</strong> option is used to allow writing of posts to file instead of
posting to a broker. The valid argument values are:</p>
<p><strong>post:</strong></p>
<p>post messages to an post_exchange</p>
<p><strong>post_broker amqp{s}://<user>:<pw>@<brokerhost>[:port]/<vhost></strong>
  <strong>post_exchange     <name>         (MANDATORY)</strong>
  <strong>on_post           <script>       (default: None)</strong></p>
<p>The <strong>post_broker</strong> defaults to the input broker if not provided.
  Just set it to another broker if you want to send the notifications
  elsewhere.</p>
<p>The <strong>post_exchange</strong> must be set by the user. This is the exchange under
  which the notifications will be posted.</p>
<p><strong>json:</strong></p>
<p>write each message to standard output, one per line in the same json format used for
  queue save/restore by the python implementation.</p>
<p><strong>url:</strong></p>
<p>just output the retrieval URL to standard output.</p>
<p>FIXME: The <strong>outlet</strong> option came from the C implementation ( <em>sr_cpump</em>  ) and it has not
been used much in the python implementation. </p>
<p>overwrite <boolean> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>overwrite</strong>  option,if set to false, avoid unnecessary downloads under these conditions :</p>
<p>1- the file to be downloaded is already on the user's file system at the right place and</p>
<p>2- the checksum of the amqp message matched the one of the file.</p>
<p>The default is False (overwrite without checking). </p>
<p>discard <boolean> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The  <strong>discard</strong>  option,if set to true, deletes the file once downloaded. This option can be
usefull when debugging or testing a configuration.</p>
<p>source_from_exchange <boolean> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>source_from_exchange</strong> option is mainly for use by administrators.
If messages is received posted directly from a source, the exchange used 
is 'xs_<brokerSourceUsername>'. Such messages be missing <em>source</em> and <em>from_cluster</em> 
headings, or a malicious user may set the values incorrectly.
To protect against both problems, administrators should set the <strong>source_from_exchange</strong> option.</p>
<p>When the option is set, values in the message for the <em>source</em> and <em>from_cluster</em> headers will then be overridden::</p>
<p>self.msg.headers['source']       = <brokerUser>
  self.msg.headers['from_cluster'] = cluster</p>
<p>replacing any values present in the message. This setting should always be used when ingesting data from a
user exchange. These fields are used to return reports to the origin of injected data.
It is commonly combined with::</p>
<pre><code>   *mirror true*
   *source_from_exchange true*
   *directory ${PBD}/${YYYYMMDD}/${SOURCE}*
</code></pre>
<p>To have data arrive in the standard format tree.</p>
<p>heartbeat <count> (default: 300 seconds)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>The <strong>heartbeat</strong> option sets how often to execute periodic processing as determined by 
the list of on_heartbeat plugins. By default, it prints a log message every heartbeat.</p>
<p>suppress_duplicates <off|on|999> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>When <strong>suppress_duplicates</strong> (also <strong>cache</strong> ) is set to a non-zero value, each new message
is compared against previous ones received, to see if it is a duplicate. If the message is 
considered a duplicate, it is skipped. What is a duplicate? A file with the same name (including 
parts header) and checksum. Every <em>hearbeat</em> interval, a cleanup process looks for files in the 
cache that have not been referenced in <strong>cache</strong> seconds, and deletes them, in order to keep 
the cache size limited. Different settings are appropriate for different use cases.</p>
<p><strong>Use of the cache is incompatible with the default <em>parts 0</em> strategy</strong>, one must specify an 
alternate strategy.  One must use either a fixed blocksize, or always never partition files. 
One must avoid the dynamic algorithm that will change the partition size used as a file grows.</p>
<p><strong>Note that the duplicate suppresion cache is local to each instance</strong>. When N 
instances share a queue, the first time a posting is received, it could be 
picked by one instance, and if a duplicate one is received it would likely 
be picked up by another instance. <strong>For effective duplicate suppression with instances</strong>, 
one must <strong>deploy two layers of subscribers</strong>. Use 
a <strong>first layer of subscribers (sr_shovels)</strong> with duplicate suppression turned 
off and output with <em>post_exchange_split</em>, which route posts by checksum to 
a <strong>second layer of subscibers (sr_winnow) whose duplicate suppression caches are active.</strong></p>
<p>kbytes_ps <count> (default: 0)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p><strong>kbytes_ps</strong> is greater than 0, the process attempts to respect this delivery
speed in kilobytes per second... ftp,ftps,or sftp)</p>
<p><strong>FIXME</strong>: kbytes_ps... only implemented by sender? or subscriber as well, data only, or messages also?</p>
<p>default_mode, default_dir_mode, preserve_modes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>Permission bits on the destination files written are controlled by the <em>preserve_mode</em> directives.
<em>preserve_modes</em> will apply the mode permissions posted by the source of the file.
If no source mode is available, the <em>default_mode</em> will be applied to files, and the
<em>default_dir_mode</em> will be applied to directories. If no default is specified,
then the operating system  defaults (on linux, controlled by umask settings)
will determine file permissions. (note that the <em>chmod</em> option is interpreted as a synonym
for <em>default_mode</em>, and <em>chmod_dir</em> is a synonym for <em>default_dir_mode</em>.)</p>
<p>recompute_chksum <boolean> (default: off)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p>For each download, the checksum is computed during transfer. If <strong>recompute_chksum</strong>
is set to True, and the recomputed checksum differ from the on in the message,
the new value will overwrite the one from the incoming amqp message. This is used
when a file is being pulled from a remote non-sarracenia source, in which case a place
holder 0 checksum is specified. On receipt, a proper checksum should be placed in the
message for downstream consumers. On can also use this method to override checksum choice.
For example, older versions of sarracenia lack SHA-512 hash support, so one could re-write
the checksums with MD5.   There are also cases, where, for various reasons, the upstream
checksums are simply wrong, and should be overridden for downstream consumers.</p>
<h2 id="delivery-completion-inflight">Delivery Completion (inflight)</h2>
<p>Failing to properly set file completion protocols is a common source of intermittent and
difficult to diagnose file transfer issues. For reliable file transfers, it is 
critical that both the sender and receiver agree on how to represent a file that isn't complete.
The <em>inflight</em> option (meaning a file is <em>in flight</em> between the sender and the receiver) supports
many protocols appropriate for different situations:</p>
<p>+--------------------------------------------------------------------------------------------+
|                                                                                            |
|               Delivery Completion Protocols (in Order of Preference)                       |
|                                                                                            |
+-------------+---------------------------------------+--------------------------------------+
| Method      | Description                           | Application                          |
+=============+=======================================+======================================+
|             |File sent with right name.             |Sending to Sarracenia, and            |
|   NONE      |Send <code>sr_post(7) &lt;sr_post.7.md&gt;</code>_     |post only when file is complete       |
|             |by AMQP after file is complete.        |                                      |
|             |                                       |(Best when available)                 |
|             | - fewer round trips (no renames)      | - Default on sr_sarra.               |
|             | - least overhead / highest speed      | - Default on sr_subscribe and sender |
|             |                                       |   when post_broker is set.           |
+-------------+---------------------------------------+--------------------------------------+
|             |Files transferred with a <em>.tmp</em> suffix.|sending to most other systems         |
| .tmp        |When complete, renamed without suffix. |(.tmp support built-in)               |
| (Suffix)    |Actual suffix is settable.             |Use to send to Sundew                 |
|             |                                       |                                      |
|             | - requires extra round trips for      |(usually a good choice)               |
|             |   rename (a little slower)            | - default when no post broker set    |
+-------------+---------------------------------------+--------------------------------------+
|             |Files transferred to a subdir.         |sending to some other systems         |
| tmp/        |When complete, renamed to parent dir.  |                                      |
| (subdir)    |Actual subdir is settable.             |                                      |
|             |                                       |                                      |
|             |same performance as Suffix method.     |                                      |
+-------------+---------------------------------------+--------------------------------------+
|             |Use Linux convention to <em>hide</em> files.  |Sending to systems that               |
| .           |Prefix names with '.'                  |do not support suffix.                |
| (Prefix)    |that need that. (compatibility)        |                                      |
|             |same performance as Suffix method.     |                                      |
+-------------+---------------------------------------+--------------------------------------+
|             |Minimum age (modification time)        |Last choice                           |
|  number     |of the file before it is considered    |guaranteed delay added                |
|  (mtime)    |complete.                              |                                      |
|             |                                       |Receiving from uncooperative          |
|             |Adds delay in every transfer.          |sources.                              |
|             |Vulnerable to network failures.        |                                      |
|             |Vulnerable to clock skew.              |(ok choice with PDS)                  |
+-------------+---------------------------------------+--------------------------------------+</p>
<p>By default ( when no <em>inflight</em> option is given ), if the post_broker is set, then a value of NONE
is used because it is assumed that it is delivering to another broker. If no post_broker
is set, the value of '.tmp' is assumed as the best option.</p>
<p>NOTES:</p>
<p>On versions of sr_sender prior to 2.18, the default was NONE, but was documented as '.tmp'
  To ensure compatibility with later versions, it is likely better to explicitly write
  the <em>inflight</em> setting.</p>
<p><em>inflight</em> was renamed from the old <em>lock</em> option in January 2017. For compatibility with
  older versions, can use <em>lock</em>, but name is deprecated.</p>
<p>The old <em>PDS</em> software (which predates MetPX Sundew) only supports FTP. The completion protocol 
  used by <em>PDS</em> was to send the file with permission 000 initially, and then chmod it to a 
  readable file. This cannot be implemented with SFTP protocol, and is not supported at all
  by Sarracenia.</p>
<p>Frequent Configuration Errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</p>
<p><strong>Setting NONE when sending to Sundew.</strong></p>
<p>The proper setting here is '.tmp'.  Without it, almost all files will get through correctly,
   but incomplete files will occasionally picked up by Sundew.  </p>
<p><strong>Using mtime method to receive from Sundew or Sarracenia:</strong></p>
<p>Using mtime is last resort. This approach injects delay and should only be used when one 
   has no influence to have the other end of the transfer use a better method. </p>
<p>mtime is vulnerable to systems whose clocks differ (causing incomplete files to be picked up.)</p>
<p>mtime is vulnerable to slow transfers, where incomplete files can be picked up because of a 
   networking issue interrupting or delaying transfers. </p>
<p><strong>Setting NONE when delivering to non-Sarracenia destination.</strong></p>
<p>NONE is to be used when there is some other means to figure out if a file is delivered.
   For example, when sending to another pump, the sender will post the announcement to 
   the destination after the file is complete, so there is no danger of it being 
   picked up early.</p>
<p>When used in-appropriately, there will occasionally be incomplete files delivered.</p>
<h1 id="periodic-processing">PERIODIC PROCESSING</h1>
<p>Most processing occurs on receipt of a message, but there is some periodic maintenance
work that happens every <em>heartbeat</em> (default is 5 minutes.)  Evey heartbeat, all of the
configured <em>on_heartbeat</em> plugins are run. By default there are three present:</p>
<ul>
<li>heartbeat_log - prints "heartbeat" in the log.</li>
<li>heartbeat_cache - ages out old entries in the cache, to minimize its size.</li>
<li>heartbeat_memory - checks the process memory usage, and restart if too big.</li>
<li>heartbeat_pulse - confirms that connectivity with brokers is still good. Restores if needed.</li>
</ul>
<p>The log will contain messages from all three plugins every heartbeat interval, and
if additional periodic processing is needed, the user can add configure addition
plugins to run with the <em>on_heartbeat</em> option. </p>
<h1 id="error-recovery">ERROR RECOVERY</h1>
<p>The tools are meant to work well un-attended, and so when transient errors occur, they do
their best to recover elegantly.  There are timeouts on all operations, and when a failure
is detected, the problem is noted for retry.  Errors can happen at many times:</p>
<ul>
<li>Establishing a connection to the broker.</li>
<li>losing a connection to the broker</li>
<li>establishing a connection to the file server for a file (for download or upload.)</li>
<li>losing a connection to the server.</li>
<li>during data transfer.</li>
</ul>
<p>Initially, the programs try to download (or send) a file a fixed number (<em>attempts</em>, default: 3) times.
If all three attempts to process the file are unsuccessful, then the file is placed in an instance's
retry file. The program then continues processing of new items. When there are no new items to
process, the program looks for a file to process in the retry queue. It then checks if the file
is so old that it is beyond the <em>retry_expire</em> (default: 2 days.) If the file is not expired, then
it triggers a new round of attempts at processing the file. If the attempts fail, it goes back
on the retry queue.</p>
<p>This algorithm ensures that programs do not get stuck on a single bad product that prevents
the rest of the queue from being processed, and allows for reasonable, gradual recovery of 
service, allowing fresh data to flow preferentially, and sending old data opportunistically
when there are gaps.</p>
<p>While fast processing of good data is very desirable, it is important to slow down when errors
start occurring. Often errors are load related, and retrying quickly will just make it worse.
Sarracenia uses exponential back-off in many points to avoid overloading a server when there
are errors. The back-off can accumulate to the point where retries could be separated by a minute
or two. Once the server begins responding normally again, the programs will return to normal
processing speed.</p>
<h1 id="examples">EXAMPLES</h1>
<p>Here is a short complete example configuration file:: </p>
<p>broker amqp://dd.weather.gc.ca/</p>
<p>subtopic model_gem_global.25km.grib2.#
  accept .*</p>
<p>This above file will connect to the dd.weather.gc.ca broker, connecting as
anonymous with password anonymous (defaults) to obtain announcements about
files in the http://dd.weather.gc.ca/model_gem_global/25km/grib2 directory.
All files which arrive in that directory or below it will be downloaded 
into the current directory (or just printed to standard output if -n option 
was specified.) </p>
<p>A variety of example configuration files are available here:</p>
<p><code>https://github.com/MetPX/sarracenia/tree/master/sarra/examples &lt;https://github.com/MetPX/sarracenia/tree/master/sarra/examples&gt;</code>_</p>
<h1 id="naming-exchanges-and-queues">NAMING EXCHANGES AND QUEUES</h1>
<p>While in most common cases, a good value is generated by the application, in some cases
there may be a need to override those choices with an explicit user specification.
To do that, one needs to be aware of the rules for naming queues:</p>
<ol>
<li>queue names start with q_</li>
<li>this is followed by <amqpUserName> (the owner/user of the queue's broker username)</li>
<li>followed by a second underscore ( _ )</li>
<li>followed by a string of the user's choice.</li>
</ol>
<p>The total length of the queue name is limited to 255 bytes of UTF-8 characters.</p>
<p>The same applies for exchanges.  The rules for those are:</p>
<ol>
<li>Exchange names start with x</li>
<li>Exchanges that end in <em>public</em> are accessible (for reading) by any authenticated user.</li>
<li>Users are permitted to create exchanges with the pattern:  xs_<amqpUserName>_<whatever> such exchanges can be written to only by that user. </li>
<li>The system (sr_audit or administrators) create the xr_<amqpUserName> exchange as a place to send reports for a given user. It is only readable by that user.</li>
<li>Administrative users (admin or feeder roles) can post or subscribe anywhere.</li>
</ol>
<p>For example, xpublic does not have xs_ and a username pattern, so it can only be posted to by admin or feeder users.
Since it ends in public, any user can bind to it to subscribe to messages posted.
Users can create exchanges such as xs_<amqpUserName>_public which can be written to by that user (by rule 3), 
and read by others (by rule 2.) A description of the conventional flow of messages through exchanges on a pump.<br />
Subscribers usually bind to the xpublic exchange to get the main data feed. this is the default in sr_subscribe.</p>
<p>Another example, a user named Alice will have at least two exchanges:</p>
<ul>
<li>xs_Alice the exhange where Alice posts her file notifications and report messages.(via many tools)</li>
<li>xr_Alice the exchange where Alice reads her report messages from (via sr_report)</li>
<li>Alice can create a new exchange by just posting to it (with sr_post or sr_cpost.) if it meets the naming rules.</li>
</ul>
<p>usually an sr_sarra run by a pump administrator will read from an exchange such as xs_Alice_mydata, 
retrieve the data corresponding to Alice��s <em>post</em> message, and make it available on the pump, 
by re-announcing it on the xpublic exchange.</p>
<h1 id="queues-and-multiple-streams">QUEUES and MULTIPLE STREAMS</h1>
<p>When executed,  <strong>sr_subscribe</strong>  chooses a queue name, which it writes
to a file named after the configuration file given as an argument to <strong>sr_subscribe</strong>
with a .queue suffix ( ."configfile".queue). 
If sr_subscribe is stopped, the posted messages continue to accumulate on the 
broker in the queue.  When the program is restarted, it uses the queuename 
stored in that file to connect to the same queue, and not lose any messages.</p>
<p>File downloads can be parallelized by running multiple sr_subscribes using
the same queue.  The processes will share the queue and each download 
part of what has been selected.  Simply launch multiple instances
of sr_subscribe in the same user/directory using the same configuration file, </p>
<p>You can also run several sr_subscribe with different configuration files to
have multiple download streams delivering into the the same directory,
and that download stream can be multi-streamed as well.</p>
<p>.. Note::</p>
<p>While the brokers keep the queues available for some time, Queues take resources on 
  brokers, and are cleaned up from time to time.  A queue which is not accessed for 
  a long (implementation dependent) period will be destroyed.  A queue which is not
  accessed and has too many (implementation defined) files queued will be destroyed.
  Processes which die should be restarted within a reasonable period of time to avoid
  loss of notifications.</p>
<h1 id="report_back-and-report_exchange">report_back and report_exchange</h1>
<p>For each download, by default, an amqp report message is sent back to the broker.
This is done with option :</p>
<ul>
<li><strong>report_back <boolean>        (default: True)</strong> </li>
<li><strong>report_exchange <report_exchangename> (default: xreport|xs_<em>username</em> )</strong></li>
</ul>
<p>When a report is generated, it is sent to the configured <em>report_exchange</em>. Administrive
components post directly to <em>xreport</em>, whereas user components post to their own 
exchanges (xs_<em>username</em>.) The report daemons then copy the messages to <em>xreport</em> after validation.</p>
<p>These reports are used for delivery tuning and for data sources to generate statistical information.
Set this option to <strong>False</strong>, to prevent generation of reports.</p>
<h1 id="logs">LOGS</h1>
<p>Components write to log files, which by default are found in ~/.cache/sarra/var/log/<component><em><config></em><instance>.log.
at the end of the day, These logs are rotated automatically by the components, and the old log gets a date suffix.
The directory in which the logs are stored can be overridden by the <strong>log</strong> option, and the number of days' logs to keep
is set by the 'logrotate' parameter.  Log files older than <strong>logrotate</strong> duration are deleted.  A duration takes a time unit suffix, such as 'd' for days, 'w' for weeks, or 'h' for hours.</p>
<ul>
<li>
<p><strong>debug</strong>  setting option debug is identical to use  <strong>loglevel debug</strong></p>
</li>
<li>
<p><strong>log</strong> the directory to store log files in.  Default value: ~/.cache/sarra/var/log (on Linux)</p>
</li>
<li>
<p><strong>logrotate</strong> duration to keep logs online, usually expressed in days ( default: 5d )</p>
</li>
<li>
<p><strong>loglevel</strong> the level of logging as expressed by python's logging.
               possible values are :  critical, error, info, warning, debug.</p>
</li>
<li>
<p><strong>chmod_log</strong> the permission bits to set on log files (default 0600 )</p>
</li>
</ul>
<p>placement is as per: <code>XDG Open Directory Specication &lt;https://specifications.freedesktop.org/basedir-spec/basedir-spec-0.6.md&gt;</code>_ ) setting the XDG_CACHE_HOME environment variable.</p>
<h1 id="instances">INSTANCES</h1>
<p>Sometimes one instance of a component and configuration is not enough to process &amp; send all available notifications.</p>
<p><strong>instances      <integer>     (default:1)</strong></p>
<p>The instance option allows launching serveral instances of a component and configuration.
When running sr_sender for example, a number of runtime files that are created.
In the ~/.cache/sarra/sender/configName directory::</p>
<p>A .sr_sender_configname.state         is created, containing the number instances.
  A .sr_sender_configname_$instance.pid is created, containing the PID  of $instance process.</p>
<p>In directory ~/.cache/sarra/var/log::</p>
<p>A .sr_sender_configname_$instance.log  is created as a log of $instance process.</p>
<p>The logs can be written in another directory than the default one with option :</p>
<p><strong>log            <directory logpath>  (default:~/.cache/sarra/var/log)</strong></p>
<p>.. note::<br />
  FIXME: indicate windows location also... dot files on windows?</p>
<p>.. Note::</p>
<p>While the brokers keep the queues available for some time, Queues take resources on 
  brokers, and are cleaned up from time to time.  A queue which is not
  accessed and has too many (implementation defined) files queued will be destroyed.
  Processes which die should be restarted within a reasonable period of time to avoid
  loss of notifications.  A queue which is not accessed for a long (implementation dependent)
  period will be destroyed. </p>
<p>.. Note::
   FIXME  The last sentence is not really right...sr_audit does track the queues'age. 
          sr_audit acts when a queue gets to the max_queue_size and not running.</p>
<h2 id="vip-activepassive-options">vip - ACTIVE/PASSIVE OPTIONS</h2>
<p><strong>sr_subscribe</strong> can be used on a single server node, or multiple nodes
could share responsibility. Some other, separately configured, high availability
software presents a <strong>vip</strong> (virtual ip) on the active server. Should
the server go down, the <strong>vip</strong> is moved on another server.
Both servers would run <strong>sr_subscribe</strong>. It is for that reason that the
following options were implemented:</p>
<ul>
<li><strong>vip          <string>          (None)</strong></li>
</ul>
<p>When you run only one <strong>sr_subscribe</strong> on one server, these options are not set,
and sr_subscribe will run in 'standalone mode'.</p>
<p>In the case of clustered brokers, you would set the options for the
moving vip.</p>
<p><strong>vip 153.14.126.3</strong></p>
<p>When <strong>sr_subscribe</strong> does not find the vip, it sleeps for 5 seconds and retries.
If it does, it consumes and process a message and than rechecks for the vip.</p>
<h1 id="posting-options">POSTING OPTIONS</h1>
<p>When advertising files downloaded for downstream consumers, one must set 
the rabbitmq configuration for an output broker.</p>
<p>The post_broker option sets all the credential information to connect to the 
output <strong>AMQP</strong> broker.</p>
<p><strong>post_broker amqp{s}://<user>:<pw>@<brokerhost>[:port]/<vhost></strong></p>
<p>Once connected to the source AMQP broker, the program builds notifications after
the download of a file has occured. To build the notification and send it to
the next hop broker, the user sets these options :</p>
<ul>
<li><strong>[--blocksize <value>]            (default: 0 (auto))</strong></li>
<li><strong>[--outlet <post|json|url>]       (default: post)</strong></li>
<li><strong>[-pbd|--post_base_dir <path>]    (optional)</strong></li>
<li><strong>post_exchange     <name>         (default: xpublic)</strong></li>
<li><strong>post_exchange_split   <number>   (default: 0)</strong></li>
<li><strong>post_base_url          <url>     (MANDATORY)</strong></li>
<li><strong>on_post           <script>       (default: None)</strong></li>
</ul>
<h2 id="-blocksize-default-0-auto">[--blocksize <value>] (default: 0 (auto))</h2>
<p>This <strong>blocksize</strong> option controls the partitioning strategy used to post files.
the value should be one of::</p>
<p>0 - autocompute an appropriate partitioning strategy (default)
   1 - always send entire files in a single part.
   <blocksize> - used a fixed partition size (example size: 1M )</p>
<p>Files can be announced as multiple parts.  Each part has a separate checksum.
The parts and their checksums are stored in the cache. Partitions can traverse
the network separately, and in paralllel.  When files change, transfers are
optimized by only sending parts which have changed.</p>
<p>The <em>outlet</em> option allows the final output to be other than a post.<br />
See <code>sr_cpump(1) &lt;sr_cpump.1.md&gt;</code>_ for details.</p>
<h2 id="-pbd-post_base_dir-optional">[-pbd|--post_base_dir <path>] (optional)</h2>
<p>The <em>post_base_dir</em> option supplies the directory path that, when combined (or found) 
in the given <em>path</em>, gives the local absolute path to the data file to be posted.
The <em>post_base_dir</em> part of the path will be removed from the posted announcement.
for sftp: url's it can be appropriate to specify a path relative to a user account.
Example of that usage would be:  -pbd ~user  -url sftp:user@host
for file: url's, base_dir is usually not appropriate.  To post an absolute path,
omit the -pbd setting, and just specify the complete path as an argument.</p>
<h2 id="post_url-mandatory">post_url <url> (MANDATORY)</h2>
<p>The <strong>post_base_url</strong> option sets how to get the file... it defines the protocol,
host, port, and optionally, the user. It is best practice to not include 
passwords in urls.</p>
<h2 id="post_exchange-default-xpublic">post_exchange <name> (default: xpublic)</h2>
<p>The <strong>post_exchange</strong> option set under which exchange the new notification
will be posted.  Im most cases it is 'xpublic'.</p>
<p>Whenever a publish happens for a product, a user can set to trigger a script.
The option <strong>on_post</strong> would be used to do such a setup.</p>
<h2 id="post_exchange_split-default-0">post_exchange_split   <number>   (default: 0)</h2>
<p>The <strong>post_exchange_split</strong> option appends a two digit suffix resulting from 
hashing the last character of the checksum to the post_exchange name,
in order to divide the output amongst a number of exchanges.  This is currently used
in high traffic pumps to allow multiple instances of sr_winnow, which cannot be
instanced in the normal way.  example::</p>
<pre><code>post_exchange_split 5
post_exchange xwinnow
</code></pre>
<p>will result in posting messages to five exchanges named: xwinnow00, xwinnow01,
xwinnow02, xwinnow03 and xwinnow04, where each exchange will receive only one fifth
of the total flow.</p>
<h2 id="remote-configurations">Remote Configurations</h2>
<p>One can specify URI's as configuration files, rather than local files. Example:</p>
<ul>
<li><strong>--config http://dd.weather.gc.ca/alerts/doc/cap.conf</strong></li>
</ul>
<p>On startup, sr_subscribe check if the local file cap.conf exists in the 
local configuration directory.  If it does, then the file will be read to find
a line like so:</p>
<ul>
<li><strong>--remote_config_url http://dd.weather.gc.ca/alerts/doc/cap.conf</strong></li>
</ul>
<p>In which case, it will check the remote URL and compare the modification time
of the remote file against the local one. The remote file is not newer, or cannot
be reached, then the component will continue with the local file.</p>
<p>If either the remote file is newer, or there is no local file, it will be downloaded, 
and the remote_config_url line will be prepended to it, so that it will continue 
to self-update in future.</p>
<h1 id="pumping">PUMPING</h1>
<p><em>This is of interest to administrators only</em></p>
<p>Sources of data need to indicate the clusters to which they would like data to be delivered.
PUMPING is implemented by administrators, and refers copying data between pumps. Pumping is
accomplished using on_message plugins which are provided with the package.</p>
<p>when messages are posted, if not destination is specified, the delivery is assumed to be 
only the pump itself.  To specify the further destination pumps for a file, sources use 
the <em>to</em> option on the post.  This option sets the to_clusters field for interpretation 
by administrators.</p>
<p>Data pumps, when ingesting data from other pumps (using shovel, subscribe or sarra components)
should include the <em>msg_to_clusters</em> plugin and specify the clusters which are reachable from
the local pump, which should have the data copied to the local pump, for further dissemination.
sample settings::</p>
<p>msg_to_clusters DDI
  msg_to_clusters DD</p>
<p>on_message msg_to_clusters</p>
<p>Given this example, the local pump (called DDI) would select messages destined for the DD or DDI clusters,
and reject those for DDSR, which isn't in the list.  This implies that there DD pump may flow
messages to the DD pump.</p>
<p>The above takes care of forward routing of messages and data to data consumers.  Once consumers
obtain data, they generate reports, and those reports need to propagate in the opposite direction,
not necessarily by the same route, back to the sources.  report routing is done using the <em>from_cluster</em>
header.  Again, this defaults to the pump where the data is injected, but may be overridden by
administrator action.</p>
<p>Administrators configure report routing shovels using the msg_from_cluster plugin. Example::</p>
<p>msg_from_cluster DDI
  msg_from_cluster DD</p>
<p>on_message msg_from_cluster</p>
<p>so that report routing shovels will obtain messages from downstream consumers and make
them available to upstream sources.</p>
<h1 id="plugin-scripts">PLUGIN SCRIPTS</h1>
<p>One can override or add functionality with python plugins scripts.
Sarracenia comes with a variety of example plugins, and uses some to implement base functionality,
such as logging (implemented by default use of msg_log, file_log, post_log plugins. )</p>
<p>Users can place their own scripts in the script sub-directory
of their config directory tree ( on Linux, the ~/.config/sarra/plugins.) </p>
<p>There are three varieties of scripts:  do_<em> and on_</em>.  Do_* scripts are used
to implement functions, adding or replacing built-in functionality, for example, to implement
additional transfer protocols.</p>
<ul>
<li>
<p>do_download - to implement additional download protocols.</p>
</li>
<li>
<p>do_get  - under ftp/ftps/http/sftp implement the get file part of the download process</p>
</li>
<li>
<p>do_poll - to implement additional polling protocols and processes.</p>
</li>
<li>
<p>do_put  - under ftp/ftps/http/sftp implement the put file part of the send process</p>
</li>
<li>
<p>do_send - to implement additional sending protocols and processes.</p>
</li>
</ul>
<p>These transfer protocol scripts should be declared using the <strong>plugin</strong> option.
Aside the targetted built-in function(s), a module <strong>registered_as</strong> that defines
a list of protocols that theses functions supports.  Exemple :</p>
<p>def registered_as(self) :
       return ['ftp','ftps']</p>
<p>Registering in such a way a plugin, if function <strong>do_download</strong> was provided in that plugin
than for any download of a message with an ftp or ftps url, it is that function that would be called.</p>
<p>On_* plugins are used more often. They allow actions to be inserted to augment the default
processing for various specialized use cases. The scripts are invoked by having a given
configuration file specify an on_<event> option. The event can be one of:</p>
<ul>
<li>
<p>plugin -- declare a set of plugins to achieve a collective function.</p>
</li>
<li>
<p>on_file -- When the reception of a file has been completed, trigger followup action.
  The <strong>on_file</strong> option defaults to file_log, which writes a downloading status message.</p>
</li>
<li>
<p>on_heartbeat -- trigger periodic followup action (every <em>heartbeat</em> seconds.)
  defaults to heatbeat_cache, and heartbeat_log.  heartbeat_cache cleans the cache periodically,
  and heartbeat_log prints a log message ( helpful in detecting the difference between problems
  and inactivity. ) </p>
</li>
<li>
<p>on_html_page -- In <strong>sr_poll</strong>, turns an html page into a python dictionary used to keep in mind
  the files already published. The package provide a working example under plugins/html_page.py.</p>
</li>
<li>
<p>on_line -- In <strong>sr_poll</strong> a line from the ls on the remote host is read in.</p>
</li>
<li>
<p>on_message -- when an sr_post(7) message has been received.  For example, a message has been received
  and additional criteria are being evaluated for download of the corresponding file.  if the on_msg
  script returns false, then it is not downloaded.  (see discard_when_lagging.py, for example,
  which decides that data that is too old is not worth downloading.)</p>
</li>
<li>
<p>on_part -- Large file transfers are split into parts.  Each part is transferred separately.
  When a completed part is received, one can specify additional processing.</p>
</li>
<li>
<p>on_post -- when a data source (or sarra) is about to post a message, permit customized
  adjustments of the post. on_part also defaults to post_log, which prints a message
  whenever a file is to be posted.</p>
</li>
<li>
<p>on_start -- runs on startup, for when a plugin needs to recover state.</p>
</li>
<li>
<p>on_stop -- runs on startup, for when a plugin needs to save state.</p>
</li>
<li>
<p>on_watch -- when the gathering of <strong>sr_watch</strong> events starts, on_watch plugin is invoked.
  It could be used to put a file in one of the watch directory and have it published when needed.</p>
</li>
</ul>
<p>The simplest example of a plugin: A do_nothing.py script for <strong>on_file</strong>::</p>
<p>class Transformer(object): 
      def <strong>init</strong>(self):
          pass</p>
<pre><code>  def on_file(self,parent):
      logger = parent.logger

      logger.info("I have no effect but adding this log line")

      return True
</code></pre>
<p>self.plugin = 'Transformer'</p>
<p>The last line of the script is specific to the kind of plugin being
written, and must be modified to correspond (on_file or an on_file, on_message 
for an on_message, etc...) The plugins stack. For example, one can have 
multiple <em>on_message</em> plugins specified, and they will be invoked in the order 
given in the configuration file.  Should one of these scripts return False, 
the processing of the message/file will stop there.  Processing will only 
continue if all configured plugins return True.  One can specify <em>on_message None</em> to 
reset the list to no plugins (removes msg_log, so it suppresses logging of message receipt.)</p>
<p>The only argument the script receives is <strong>parent</strong>, which is a data
structure containing all the settings, as <strong>parent.<setting></strong>, and
the content of the message itself as <strong>parent.msg</strong> and the headers
are available as <strong>parent.msg[ <header> ]</strong>.  The path to write a file
to is available as There is also <strong>parent.msg.new_dir</strong> / <strong>parent.msg.new_file</strong></p>
<p>There is also registered plugins used to add or overwrite built-in 
transfer protocol scripts. They should be declared using the <strong>plugin</strong> option.
They must register the protocol (url scheme) that they indent to provide services for.
The script for transfer protocols are :</p>
<ul>
<li>
<p>do_download - to implement additional download protocols.</p>
</li>
<li>
<p>do_get  - under ftp/ftps/http/sftp implement the get part of the download process</p>
</li>
<li>
<p>do_poll - to implement additional polling protocols and processes.</p>
</li>
<li>
<p>do_put  - under ftp/ftps/http/sftp implement the put part of the send process</p>
</li>
<li>
<p>do_send - to implement additional sending protocols and processes.</p>
</li>
</ul>
<p>The registration is done with a module named <strong>registered_as</strong> . It defines
a list of protocols that the provided module supports.</p>
<p>The simplest example of a plugin: A do_nothing.py script for <strong>on_file</strong>::</p>
<p>class Transformer(object): 
      def <strong>init</strong>(self):
          pass</p>
<pre><code>  def on_put(self,parent):
      msg = parent.msg

      if ':' in msg.relpath : return None

      netloc = parent.destination.replace("sftp://",'')
      if netloc[-1] == '/' : netloc = netloc[:-1]

      cmd = '/usr/bin/scp ' + msg.relpath + ' ' +  netloc + ':' + msg.new_dir + os.sep + msg.new_file

      status, answer = subprocess.getstatusoutput(cmd)

      if status == 0 : return True

      return False

  def registered_as(self) :
      return ['sftp']
</code></pre>
<p>self.plugin = 'Transformer'</p>
<p>This plugin registers for sftp. A sender with such a plugin would put the product using scp.
It would be confusing for scp to have the source path with a ':' in the filename... Here the
case is handled by returning None and letting python sending the file over. The <strong>parent</strong>
argument holds all the needed program informations.
Some other available variables::</p>
<p>parent.msg.new_file     :  name of the file to write.
  parent.msg.new_dir      :  name of the directory in which to write the file.
  parent.msg.local_offset :  offset position in the local file
  parent.msg.offset       :  offset position of the remote file
  parent.msg.length       :  length of file or part
  parent.msg.in_partfile  :  T/F file temporary in part file
  parent.msg.local_url    :  url for reannouncement</p>
<p>See the <code>Programming Guide &lt;Prog.md&gt;</code>_ for more details.</p>
<h1 id="queue-saverestore">Queue Save/Restore</h1>
<h2 id="sender-destination-unavailable">Sender Destination Unavailable</h2>
<p>If the server to which the files are being sent is going to be unavailable for
a prolonged period, and there is a large number of messages to send to them, then
the queue will build up on the broker. As the performance of the entire broker
is affected by large queues, one needs to minimize such queues.</p>
<p>The <em>-save</em> and <em>-restore</em> options are used get the messages away from the broker
when too large a queue will certainly build up.
The <em>-save</em> option copies the messages to a (per instance) disk file (in the same directory
that stores state and pid files), as json encoded strings, one per line.
When a queue is building up::</p>
<p>sr_sender stop <config> 
   sr_sender -save start <config> </p>
<p>And run the sender in <em>save</em> mode (which continually writes incoming messages to disk)
in the log, a line for each message written to disk::</p>
<p>2017-03-03 12:14:51,386 [INFO] sr_sender saving 2 message topic: v02.post.home.peter.sarra_devdocroot.sub.SASP34_LEMM_031630__LEDA_60215</p>
<p>Continue in this mode until the absent server is again available.  At that point::</p>
<p>sr_sender stop <config> 
   sr_sender -restore start <config> </p>
<p>While restoring from the disk file, messages like the following will appear in the log::</p>
<p>2017-03-03 12:15:02,969 [INFO] sr_sender restoring message 29 of 34: topic: v02.post.home.peter.sarra_devdocroot.sub.ON_02GD022_daily_hydrometric.csv</p>
<p>After the last one::</p>
<p>2017-03-03 12:15:03,112 [INFO] sr_sender restore complete deleting save file: /home/peter/.cache/sarra/sender/tsource2send/sr_sender_tsource2send_0000.save </p>
<p>and the sr_sender will function normally thereafter.</p>
<h2 id="shovel-saverestore">Shovel Save/Restore</h2>
<p>If a queue builds up on a broker because a subscriber is unable to process
messages, overall broker performance will suffer, so leaving the queue lying around
is a problem. As an administrator, one could keep a configuration like this
around::</p>
<p>% more ~/tools/save.conf
  broker amqp://tfeed@localhost/
  topic_prefix v02.post
  exchange xpublic</p>
<p>post_rate_limit 50
  on_post post_rate_limit
  post_broker amqp://tfeed@localhost/</p>
<p>The configuration relies on the use of an administrator or feeder account.
note the queue which has messages in it, in this case q_tsub.sr_subscribe.t.99524171.43129428.  Invoke the shovel in save mode to consumer messages from the queue
and save them to disk::</p>
<p>% cd ~/tools
  % sr_shovel -save -queue q_tsub.sr_subscribe.t.99524171.43129428 foreground save.conf</p>
<p>2017-03-18 13:07:27,786 [INFO] sr_shovel start
  2017-03-18 13:07:27,786 [INFO] sr_sarra run
  2017-03-18 13:07:27,786 [INFO] AMQP  broker(localhost) user(tfeed) vhost(/)
  2017-03-18 13:07:27,788 [WARNING] non standard queue name q_tsub.sr_subscribe.t.99524171.43129428
  2017-03-18 13:07:27,788 [INFO] Binding queue q_tsub.sr_subscribe.t.99524171.43129428 with key v02.post.# from exchange xpublic on broker amqp://tfeed@localhost/
  2017-03-18 13:07:27,790 [INFO] report_back to tfeed@localhost, exchange: xreport
  2017-03-18 13:07:27,792 [INFO] sr_shovel saving to /home/peter/.cache/sarra/shovel/save/sr_shovel_save_0000.save for future restore
  2017-03-18 13:07:27,794 [INFO] sr_shovel saving 1 message topic: v02.post.observations.swob-ml.20170318.CPSL.2017-03-18-1600-CPSL-AUTO-swob.xml
  2017-03-18 13:07:27,795 [INFO] sr_shovel saving 2 message topic: v02.post.hydrometric.doc.hydrometric_StationList.csv
          .
          .
          .
  2017-03-18 13:07:27,901 [INFO] sr_shovel saving 188 message topic: v02.post.hydrometric.csv.ON.hourly.ON_hourly_hydrometric.csv
  2017-03-18 13:07:27,902 [INFO] sr_shovel saving 189 message topic: v02.post.hydrometric.csv.BC.hourly.BC_hourly_hydrometric.csv</p>
<p>^C2017-03-18 13:11:27,261 [INFO] signal stop
  2017-03-18 13:11:27,261 [INFO] sr_shovel stop</p>
<p>% wc -l /home/peter/.cache/sarra/shovel/save/sr_shovel_save_0000.save
  189 /home/peter/.cache/sarra/shovel/save/sr_shovel_save_0000.save
  % </p>
<p>The messages are written to a file in the caching directory for future use, with
the name of the file being based on the configuration name used.   the file is in
json format, one message per line (lines are very long.) and so filtering with other tools
is possible to modify the list of saved messages.  Note that a single save file per
configuration is automatically set, so to save multiple queues, one would need one configurations
file per queue to be saved.  Once the subscriber is back in service, one can return the messages
saved to a file into the same queue::</p>
<p>% sr_shovel -restore_to_queue q_tsub.sr_subscribe.t.99524171.43129428 foreground save.conf</p>
<p>2017-03-18 13:15:33,610 [INFO] sr_shovel start
  2017-03-18 13:15:33,611 [INFO] sr_sarra run
  2017-03-18 13:15:33,611 [INFO] AMQP  broker(localhost) user(tfeed) vhost(/)
  2017-03-18 13:15:33,613 [INFO] Binding queue q_tfeed.sr_shovel.save with key v02.post.# from exchange xpublic on broker amqp://tfeed@localhost/
  2017-03-18 13:15:33,615 [INFO] report_back to tfeed@localhost, exchange: xreport
  2017-03-18 13:15:33,618 [INFO] sr_shovel restoring 189 messages from save /home/peter/.cache/sarra/shovel/save/sr_shovel_save_0000.save 
  2017-03-18 13:15:33,620 [INFO] sr_shovel restoring message 1 of 189: topic: v02.post.observations.swob-ml.20170318.CPSL.2017-03-18-1600-CPSL-AUTO-swob.xml
  2017-03-18 13:15:33,620 [INFO] msg_log received: 20170318165818.878 http://localhost:8000/ observations/swob-ml/20170318/CPSL/2017-03-18-1600-CPSL-AUTO-swob.xml topic=v02.post.observations.swob-ml.20170318.CPSL.2017-03-18-1600-CPSL-AUTO-swob.xml lag=1034.74 sundew_extension=DMS:WXO_RENAMED_SWOB:MSC:XML::20170318165818 source=metpx mtime=20170318165818.878 sum=d,66f7249bd5cd68b89a5ad480f4ea1196 to_clusters=DD,DDI.CMC,DDI.EDM,DDI.CMC,CMC,SCIENCE,EDM parts=1,5354,1,0,0 toolong=1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890�� from_cluster=DD atime=20170318165818.878 filename=2017-03-18-1600-CPSL-AUTO-swob.xml 
     .
     .
     .
  2017-03-18 13:15:33,825 [INFO] post_log notice=20170318165832.323 http://localhost:8000/hydrometric/csv/BC/hourly/BC_hourly_hydrometric.csv headers={'sundew_extension': 'BC:HYDRO:CSV:DEV::20170318165829', 'toolong': '1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890������1234567890��', 'filename': 'BC_hourly_hydrometric.csv', 'to_clusters': 'DD,DDI.CMC,DDI.EDM,DDI.CMC,CMC,SCIENCE,EDM', 'sum': 'd,a22b2df5e316646031008654b29c4ac3', 'parts': '1,12270407,1,0,0', 'source': 'metpx', 'from_cluster': 'DD', 'atime': '20170318165832.323', 'mtime': '20170318165832.323'}
  2017-03-18 13:15:33,826 [INFO] sr_shovel restore complete deleting save file: /home/peter/.cache/sarra/shovel/save/sr_shovel_save_0000.save </p>
<p>2017-03-18 13:19:26,991 [INFO] signal stop
  2017-03-18 13:19:26,991 [INFO] sr_shovel stop
  % </p>
<p>All the messages saved are returned to the named <em>return_to_queue</em>. Note that the use of the <em>post_rate_limit</em>
plugin prevents the queue from being flooded with hundreds of messages per second. The rate limit to use will need
to be tuned in practice.</p>
<p>by default the file name for the save file is chosen to be in ~/.cache/sarra/shovel/<config>_<instance>.save.
To Choose a different destination, <em>save_file</em> option is available::</p>
<p>sr_shovel -save_file <code>pwd</code>/here -restore_to_queue q_tsub.sr_subscribe.t.99524171.43129428 ./save.conf foreground</p>
<p>will create the save files in the current directory named here_000x.save where x is the instance number (0 for foreground.)</p>
<h1 id="roles-feederadmindeclare">ROLES - feeder/admin/declare</h1>
<p><em>of interest only to administrators</em></p>
<p>Administrative options are set using::</p>
<p>sr_subscribe edit admin</p>
<p>The <em>feeder</em> option specifies the account used by default system transfers for components such as
sr_shovel, sr_sarra and sr_sender (when posting).</p>
<ul>
<li>
<p><strong>feeder    amqp{s}://<user>:<pw>@<post_brokerhost>[:port]/<vhost></strong></p>
</li>
<li>
<p><strong>admin   <name>        (default: None)</strong></p>
</li>
</ul>
<p>When set, the admin option will cause sr start to start up the sr_audit daemon.
FIXME: current versions, all users run sr_audit to notice dead subscribers.
Most users are defined using the <em>declare</em> option.</p>
<ul>
<li><strong>declare <role> <name>   (no defaults)</strong></li>
</ul>
<h2 id="subscriber">subscriber</h2>
<p>A subscriber is user that can only subscribe to data and return report messages. Subscribers are
  not permitted to inject data.  Each subscriber has an xs_<user> named exchange on the pump,
  where if a user is named <em>Acme</em>, the corresponding exchange will be <em>xs_Acme</em>.  This exchange
  is where an sr_subscribe process will send it's report messages.</p>
<p>By convention/default, the <em>anonymous</em> user is created on all pumps to permit subscription without
  a specific account.</p>
<h2 id="source">source</h2>
<p>A user permitted to subscribe or originate data.  A source does not necessarily represent
  one person or type of data, but rather an organization responsible for the data produced.
  So if an organization gathers and makes available ten kinds of data with a single contact
  email or phone number for questions about the data and it's availability, then all of
  those collection activities might use a single 'source' account.</p>
<p>Each source gets a xs_<user> exchange for injection of data posts, and, similar to a subscriber
  to send report messages about processing and receipt of data. source may also have an xl_<user>
  exchange where, as per report routing configurations, report messages of consumers will be sent.</p>
<p>User credentials are placed in the credentials files, and <em>sr_audit</em> will update
the broker to accept what is specified in that file, as long as the admin password is
already correct.</p>
<h1 id="configuration-files">CONFIGURATION FILES</h1>
<p>While one can manage configuration files using the <em>add</em>, <em>remove</em>,
<em>list</em>, <em>edit</em>, <em>disable</em>, and <em>enable</em> actions, one can also do all
of the same activities manually by manipulating files in the settings
directory.  The configuration files for an sr_subscribe configuration 
called <em>myflow</em> would be here:</p>
<ul>
<li>
<p>linux: ~/.config/sarra/subscribe/myflow.conf (as per: <code>XDG Open Directory Specication &lt;https://specifications.freedesktop.org/basedir-spec/basedir-spec-0.6.md&gt;</code>_ ) </p>
</li>
<li>
<p>Windows: %AppDir%/science.gc.ca/sarra/myflow.conf , this might be:
   C:\Users\peter\AppData\Local\science.gc.ca\sarra\myflow.conf</p>
</li>
<li>
<p>MAC: FIXME.</p>
</li>
</ul>
<p>The top of the tree has  <em>~/.config/sarra/default.conf</em> which contains settings that
are read as defaults for any component on start up.  in the same directory, <em>~/.config/sarra/credentials.conf</em> contains credentials (passwords) to be used by sarracenia ( <code>CREDENTIALS</code>_ for details. )</p>
<p>One can also set the XDG_CONFIG_HOME environment variable to override default placement, or 
individual configuration files can be placed in any directory and invoked with the 
complete path.   When components are invoked, the provided file is interpreted as a 
file path (with a .conf suffix assumed.)  If it is not found as a file path, then the 
component will look in the component's config directory ( <strong>config_dir</strong> / <strong>component</strong> )
for a matching .conf file.</p>
<p>If it is still not found, it will look for it in the site config dir
(linux: /usr/share/default/sarra/<strong>component</strong>).</p>
<p>Finally, if the user has set option <strong>remote_config</strong> to True and if he has
configured web sites where configurations can be found (option <strong>remote_config_url</strong>),
The program will try to download the named file from each site until it finds one.
If successful, the file is downloaded to <strong>config_dir/Downloads</strong> and interpreted
by the program from there.  There is a similar process for all <em>plugins</em> that can
be interpreted and executed within sarracenia components.  Components will first
look in the <em>plugins</em> directory in the users config tree, then in the site
directory, then in the sarracenia package itself, and finally it will look remotely.</p>
<h1 id="see-also">SEE ALSO</h1>
<p><strong>User Commands:</strong></p>
<p><code>sr_subscribe(1) &lt;sr_subscribe.1.md&gt;</code>_ - Select and Conditionally Download Posted Files</p>
<p><code>sr_post(1) &lt;sr_post.1.md&gt;</code>_ - post announcemensts of specific files.</p>
<p><code>sr_watch(1) &lt;sr_watch.1.md&gt;</code>_ - post that loops, watching over directories.</p>
<p><code>sr_sender(1) &lt;sr_sender.1.md&gt;</code>_ - subscribes to messages pointing at local files, and sends them to remote systems and reannounces them there.</p>
<p><code>sr_report(1) &lt;sr_report.1.md&gt;</code>_ - process report messages.</p>
<p><strong>Pump Adminisitrator Commands:</strong></p>
<p><code>sr_shovel(8) &lt;sr_shovel.8.md&gt;</code>_ - process messages (no downloading.)</p>
<p><code>sr_winnow(8) &lt;sr_winnow.8.md&gt;</code>_ - a shovel with cache on, to winnow wheat from chaff.</p>
<p><code>sr_sarra(8) &lt;sr_sarra.8.md&gt;</code>_ - Subscribe, Acquire, and ReAdvertise tool.</p>
<p><code>sr_audit(8) &lt;sr_audit.8.md&gt;</code>_ - Monitoring daemon, audits running configurations, restarts missing instances.</p>
<p><code>sr_log2save(8) &lt;sr_log2save.8.md&gt;</code>_ - Convert logfile lines to .save Format for reload/resend.</p>
<p><strong>Formats:</strong></p>
<p><code>sr_post(7) &lt;sr_post.7.md&gt;</code>_ - The format of announcement messages.</p>
<p><code>sr_report(7) &lt;sr_report.7.md&gt;</code>_ - the format of report messages.</p>
<p><code>sr_pulse(7) &lt;sr_pulse.7.md&gt;</code>_ - The format of pulse messages.</p>
<p><strong>Home Page:</strong></p>
<p><code>https://github.com/MetPX/ &lt;https://github.com/MetPX&gt;</code>_ - sr_subscribe is a component of MetPX-Sarracenia, the AMQP based data pump.</p>
<h1 id="sundew-compatibility-options">SUNDEW COMPATIBILITY OPTIONS</h1>
<p>For compatibility with sundew, there are some additional delivery options which can be specified.</p>
<h2 id="destfn_script-defaultnone">destfn_script <script> (default:None)</h2>
<p>This option defines a script to be run when everything is ready
for the delivery of the product.  The script receives the sr_sender class
instance.  The script takes the parent as an argument, and for example, any
modification to  <strong>parent.msg.new_file</strong>  will change the name of the file written locally.</p>
<h2 id="filename-defaultwhatfn">filename <keyword> (default:WHATFN)</h2>
<p>From <strong>metpx-sundew</strong> the support of this option give all sorts of possibilities
for setting the remote filename. Some <strong>keywords</strong> are based on the fact that
<strong>metpx-sundew</strong> filenames are five (to six) fields strings separated by for colons.
The possible keywords are :</p>
<p><strong>WHATFN</strong>
 - the first part of the sundew filename (string before first :)</p>
<p><strong>HEADFN</strong>
 - HEADER part of the sundew filename</p>
<p><strong>SENDER</strong>
 - the sundew filename may end with a string SENDER=<string> in this case the <string> will be the remote filename</p>
<p><strong>NONE</strong>
 - deliver with the complete sundew filename (without :SENDER=...)</p>
<p><strong>NONESENDER</strong>
 - deliver with the complete sundew filename (with :SENDER=...)</p>
<p><strong>TIME</strong>
 - time stamp appended to filename. Example of use: WHATFN:TIME</p>
<p><strong>DESTFN=str</strong>
 - direct filename declaration str</p>
<p><strong>SATNET=1,2,3,A</strong>
 - cmc internal satnet application parameters</p>
<p><strong>DESTFNSCRIPT=script.py</strong>
 - invoke a script (same as destfn_script) to generate the name of the file to write</p>
<p><strong>accept <regexp pattern> [<keyword>]</strong></p>
<p>keyword can be added to the <strong>accept</strong> option. The keyword is any one of the <strong>filename</strong>
tion.  A message that matched against the accept regexp pattern, will have its remote_file
plied this keyword option.  This keyword has priority over the preceeding <strong>filename</strong> one.</p>
<p>The <strong>regexp pattern</strong> can be use to set directory parts if part of the message is put
to parenthesis. <strong>sr_sender</strong> can use these parts to build the directory name. The
rst enclosed parenthesis strings will replace keyword <strong>${0}</strong> in the directory name...
the second <strong>${1}</strong> etc.</p>
<p>example of use::</p>
<pre><code>  filename NONE

  directory /this/first/target/directory

  accept .*file.*type1.*

  directory /this/target/directory

  accept .*file.*type2.*

  accept .*file.*type3.*  DESTFN=file_of_type3

  directory /this/${0}/pattern/${1}/directory

  accept .*(2016....).*(RAW.*GRIB).*
</code></pre>
<p>A selected message by the first accept would be delivered unchanged to the first directory.</p>
<p>A selected message by the second accept would be delivered unchanged to the second directory.</p>
<p>A selected message by the third accept would be renamed "file_of_type3" in the second directory.</p>
<p>A selected message by the forth accept would be delivered unchanged to a directory.</p>
<p>named  <em>/this/20160123/pattern/RAW_MERGER_GRIB/directory</em> if the message would have a notice like:</p>
<p><strong>20150813161959.854 http://this.pump.com/ relative/path/to/20160123_product_RAW_MERGER_GRIB_from_CMC</strong></p>
<h2 id="field-replacements">Field Replacements</h2>
<p>In MetPX Sundew, there is a much more strict file naming standard, specialised for use with 
World Meteorological Organization (WMO) data.   Note that the file naming convention predates, and 
bears no relation to the WMO file naming convention currently approved, but is strictly an internal 
format.   The files are separated into six fields by colon characters.  The first field, DESTFN, 
gives the WMO (386 style) Abbreviated Header Line (AHL) with underscores replacing blanks::</p>
<p>TTAAii CCCC YYGGGg BBB ...  </p>
<p>(see WMO manuals for details) followed by numbers to render the product unique (as in practice, 
though not in theory, there are a large number of products which have the same identifiers.)
The meanings of the fifth field is a priority, and the last field is a date/time stamp.<br />
The other fields vary in meaning depending on context.  A sample file name::</p>
<p>SACN43_CWAO_012000_AAA_41613:ncp1:CWAO:SA:3.A.I.E:3:20050201200339</p>
<p>If a file is sent to sarracenia and it is named according to the sundew conventions, then the 
following substition fields are available::</p>
<p>${T1}    replace by bulletin's T1
  ${T2}    replace by bulletin's T2
  ${A1}    replace by bulletin's A1
  ${A2}    replace by bulletin's A2
  ${ii}    replace by bulletin's ii
  ${CCCC}  replace by bulletin's CCCC
  ${YY}    replace by bulletin's YY   (obs. day)
  ${GG}    replace by bulletin's GG   (obs. hour)
  ${Gg}    replace by bulletin's Gg   (obs. minute)
  ${BBB}   replace by bulletin's bbb
  ${RYYYY} replace by reception year
  ${RMM}   replace by reception month
  ${RDD}   replace by reception day
  ${RHH}   replace by reception hour
  ${RMN}   replace by reception minutes
  ${RSS}   replace by reception second</p>
<p>The 'R' fields from from the sixth field, and the others come from the first one.
When data is injected into sarracenia from Sundew, the <em>sundew_extension</em> message header
will provide the source for these substitions even if the fields have been removed
from the delivered file names.</p>
<h2 id="deprecated-settings">DEPRECATED SETTINGS</h2>
<p>These settings pertain to previous versions of the client, and have been superceded.</p>
<ul>
<li><strong>host          <broker host>  (unsupported)</strong> </li>
<li><strong>amqp-user     <broker user>  (unsupported)</strong> </li>
<li><strong>amqp-password <broker pass>  (unsupported)</strong> </li>
<li><strong>http-user     <url    user>  (now in credentials.conf)</strong> </li>
<li><strong>http-password <url    pass>  (now in credentials.conf)</strong> </li>
<li><strong>topic         <amqp pattern> (deprecated)</strong> </li>
<li><strong>exchange_type <type>         (default: topic)</strong> </li>
<li><strong>exchange_key  <amqp pattern> (deprecated)</strong> </li>
<li><strong>lock      <locktext>         (renamed to inflight)</strong> </li>
</ul>
<h1 id="history">HISTORY</h1>
<p>Dd_subscribe was initially developed for  <strong>dd.weather.gc.ca</strong>, an Environment Canada website 
where a wide variety of meteorological products are made available to the public. It is from
the name of this site that the sarracenia suite takes the dd_ prefix for it's tools.  The initial
version was deployed in 2013 on an experimental basis.  The following year, support of checksums
was added, and in the fall of 2015, the feeds were updated to v02.  dd_subscribe still works,
but it uses the deprecated settings described above.  It is implemented python2, whereas
the sarracenia toolkit is in python3.</p>
<p>In 2007, when the MetPX was originally open sourced, the staff responsible were part of
Environment Canada.  In honour of the Species At Risk Act (SARA), to highlight the plight
of disappearing species which are not furry (the furry ones get all the attention) and
because search engines will find references to names which are more unusual more easily, 
the original MetPX WMO switch was named after a carnivorous plant on the Species At
Risk Registry:  The <em>Thread-leaved Sundew</em>.  </p>
<p>The organization behind Metpx have since moved to Shared Services Canada, but when
it came time to name a new module, we kept with a theme of carnivorous plants, and 
chose another one indigenous to some parts of Canada: <em>Sarracenia</em> any of a variety
of insectivorous pitcher plants. We like plants that eat meat!  </p>
<h2 id="dd_subscribe-renaming">dd_subscribe Renaming</h2>
<p>The new module (MetPX-Sarracenia) has many components, is used for more than 
distribution, and more than one web site, and causes confusion for sys-admins thinking
it is associated with the dd(1) command (to convert and copy files).  So, we switched
all the components to use the sr_ prefix.</p>
